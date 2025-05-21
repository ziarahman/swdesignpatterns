# Design Patterns: Concurrency - Thread Pool

## Thread Pool Pattern - Simple Explanation

**SITUATION: YOU'RE EXECUTING PARALLEL TASKS**  
You have multiple tasks to execute concurrently.

Initially, you have:

#### Java
```java
class TaskExecutor {
    void execute(Runnable task) {
        new Thread(task).start();
    }
}
``` 
All good.

**THEN THE TASKS MULTIPLY**  
Now you have:
- Many concurrent tasks
- Resource exhaustion
- Thread creation overhead
- No task prioritization
- Uncontrolled concurrency

You're stuck with:
```java
for (Task task : tasks) {
    new Thread(() -> {
        try {
            task.execute();
        } catch (Exception e) {
            // Handle error
        }
    }).start();
}
``` 
Thread explosion!

**WHAT'S THE PROBLEM?**  
- Too many threads created
- Resource waste
- Performance degradation
- No task management
- No thread reuse

**HOW THREAD POOL SAVES YOU**  
Create a pool of reusable worker threads.

You say: "Execute these tasks with your worker threads." 

And the Thread Pool:
- Manages thread lifecycle
- Queues tasks
- Reuses threads
- Controls concurrency
- Handles errors

Now you just:
```java
ThreadPool pool = new ThreadPool(10);
pool.submit(() -> processTask());
pool.submit(() -> processAnotherTask());
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java
```java
public class ThreadPool {
    private final BlockingQueue<Runnable> taskQueue;
    private final List<WorkerThread> workers;
    private final int poolSize;
    private volatile boolean running = true;
    
    public ThreadPool(int poolSize) {
        this.poolSize = poolSize;
        this.taskQueue = new LinkedBlockingQueue<>();
        this.workers = new ArrayList<>(poolSize);
        
        for (int i = 0; i < poolSize; i++) {
            WorkerThread worker = new WorkerThread();
            workers.add(worker);
            worker.start();
        }
    }
    
    public void submit(Runnable task) {
        if (running) {
            taskQueue.offer(task);
        }
    }
    
    public void shutdown() {
        running = false;
        workers.forEach(WorkerThread::interrupt);
    }
    
    private class WorkerThread extends Thread {
        @Override
        public void run() {
            while (running) {
                try {
                    Runnable task = taskQueue.take();
                    task.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
        }
    }
}

// Usage
class TaskProcessor {
    private final ThreadPool pool;
    
    public TaskProcessor(int poolSize) {
        this.pool = new ThreadPool(poolSize);
    }
    
    public void process(List<Task> tasks) {
        tasks.forEach(task -> 
            pool.submit(() -> task.execute()));
    }
}
```

#### TypeScript
```typescript
class ThreadPool {
    private taskQueue: Array<() => Promise<void>>;
    private workers: Worker[];
    private running: boolean;
    
    constructor(private poolSize: number) {
        this.taskQueue = [];
        this.workers = [];
        this.running = true;
        
        for (let i = 0; i < poolSize; i++) {
            this.addWorker();
        }
    }
    
    private addWorker(): void {
        const worker = new Worker(async () => {
            while (this.running) {
                const task = await this.getNextTask();
                if (task) {
                    try {
                        await task();
                    } catch (error) {
                        console.error('Task error:', error);
                    }
                }
            }
        });
        
        this.workers.push(worker);
    }
    
    private async getNextTask(): Promise<(() => Promise<void>) | null> {
        while (this.running && this.taskQueue.length === 0) {
            await new Promise(resolve => setTimeout(resolve, 100));
        }
        return this.taskQueue.shift() || null;
    }
    
    async submit(task: () => Promise<void>): Promise<void> {
        if (this.running) {
            this.taskQueue.push(task);
        }
    }
    
    shutdown(): void {
        this.running = false;
    }
}

// Usage
class TaskProcessor {
    private pool: ThreadPool;
    
    constructor(poolSize: number) {
        this.pool = new ThreadPool(poolSize);
    }
    
    async processTasks(tasks: Task[]): Promise<void> {
        await Promise.all(
            tasks.map(task => 
                this.pool.submit(async () => task.execute()))
        );
    }
}
```

#### Python
```python
from queue import Queue
from threading import Thread
from typing import Callable, List
import time

class ThreadPool:
    def __init__(self, pool_size: int):
        self.task_queue: Queue = Queue()
        self.workers: List[Thread] = []
        self.running = True
        
        for _ in range(pool_size):
            worker = Thread(target=self._worker_loop)
            worker.daemon = True
            worker.start()
            self.workers.append(worker)
    
    def _worker_loop(self):
        while self.running:
            try:
                task = self.task_queue.get(timeout=1.0)
                try:
                    task()
                except Exception as e:
                    print(f"Task error: {e}")
                finally:
                    self.task_queue.task_done()
            except:
                continue
    
    def submit(self, task: Callable[[], None]):
        if self.running:
            self.task_queue.put(task)
    
    def shutdown(self):
        self.running = False
        for worker in self.workers:
            worker.join()

# Usage
class TaskProcessor:
    def __init__(self, pool_size: int):
        self.pool = ThreadPool(pool_size)
    
    def process_tasks(self, tasks: List[Task]):
        for task in tasks:
            self.pool.submit(task.execute)
        self.pool.task_queue.join()
```

#### Go
```go
package main

import (
    "context"
    "sync"
)

type ThreadPool struct {
    taskQueue  chan func()
    wg         sync.WaitGroup
    ctx        context.Context
    cancel     context.CancelFunc
}

func NewThreadPool(poolSize int) *ThreadPool {
    ctx, cancel := context.WithCancel(context.Background())
    pool := &ThreadPool{
        taskQueue: make(chan func(), 100),
        ctx:       ctx,
        cancel:    cancel,
    }
    
    for i := 0; i < poolSize; i++ {
        pool.wg.Add(1)
        go pool.worker()
    }
    
    return pool
}

func (p *ThreadPool) worker() {
    defer p.wg.Done()
    
    for {
        select {
        case task := <-p.taskQueue:
            task()
        case <-p.ctx.Done():
            return
        }
    }
}

func (p *ThreadPool) Submit(task func()) {
    select {
    case p.taskQueue <- task:
    case <-p.ctx.Done():
    }
}

func (p *ThreadPool) Shutdown() {
    p.cancel()
    p.wg.Wait()
    close(p.taskQueue)
}

// Usage
type TaskProcessor struct {
    pool *ThreadPool
}

func NewTaskProcessor(poolSize int) *TaskProcessor {
    return &TaskProcessor{
        pool: NewThreadPool(poolSize),
    }
}

func (p *TaskProcessor) ProcessTasks(tasks []Task) {
    var wg sync.WaitGroup
    for _, task := range tasks {
        wg.Add(1)
        t := task // Capture for closure
        p.pool.Submit(func() {
            defer wg.Done()
            t.Execute()
        })
    }
    wg.Wait()
}
```

**Key Differences:**
- Java uses BlockingQueue for task management
- TypeScript implements with async/await and Promises
- Python uses Queue and daemon threads
- Go leverages goroutines and channels

**WHEN SHOULD YOU USE IT?**
- When executing many parallel tasks
- When managing thread lifecycle
- When controlling resource usage
- When implementing work queues
- When optimizing thread creation

**WHERE YOU'VE ALREADY SEEN IT**
- Web servers
- Database connection pools
- Task schedulers
- Background job processors
- Application servers
