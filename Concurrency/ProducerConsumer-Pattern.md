# Design Patterns: Concurrency - Producer Consumer

## Producer-Consumer Pattern - Simple Explanation

**SITUATION: YOU'RE HANDLING ASYNC TASKS**  
You have tasks being generated and processed. 

Initially, you have: 

#### Java
```java
void processTask(Task task) {
    processImmediately(task);
}
``` 
All good.

**THEN THE REQUIREMENTS CHANGE**  
Now you have:  
- Tasks coming in faster than processing
- Limited processing resources
- Need for task buffering
- Multiple producers and consumers

You're stuck with: 
```java
List<Task> tasks = new ArrayList<>();
// Producer
void addTask(Task task) {
    tasks.add(task);  // Could overflow!
}
// Consumer
void processNext() {
    if (!tasks.isEmpty()) {  // Race condition!
        Task task = tasks.remove(0);
        process(task);
    }
}
``` 
Race conditions and memory issues!

**WHAT'S THE PROBLEM?**  
- No control over task generation rate
- Risk of out-of-memory errors
- Race conditions in shared queue
- No coordination between producers and consumers

**HOW PRODUCER-CONSUMER SAVES YOU**  
Create a bounded buffer between producers and consumers.

You say: "Put tasks in the queue, workers will process them when ready." 

And the Pattern:  
- Manages a bounded buffer
- Synchronizes access
- Handles backpressure
- Coordinates workers

Now you just: 
```java
BlockingQueue<Task> queue = new ArrayBlockingQueue<>(100);
// Producer
queue.put(task);  // Blocks if queue is full
// Consumer
Task task = queue.take();  // Blocks if queue is empty
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
class ProducerConsumer {
    private final BlockingQueue<Task> queue;
    private final int nConsumers;
    private final ExecutorService executorService;
    
    public ProducerConsumer(int capacity, int nConsumers) {
        this.queue = new ArrayBlockingQueue<>(capacity);
        this.nConsumers = nConsumers;
        this.executorService = Executors.newFixedThreadPool(nConsumers);
    }
    
    public void startConsumers() {
        for (int i = 0; i < nConsumers; i++) {
            executorService.submit(this::consumeTasks);
        }
    }
    
    public void produce(Task task) throws InterruptedException {
        queue.put(task);  // Blocks if queue is full
    }
    
    private void consumeTasks() {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                Task task = queue.take();  // Blocks if queue is empty
                processTask(task);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }
    
    private void processTask(Task task) {
        // Process the task
    }
    
    public void shutdown() {
        executorService.shutdown();
    }
}
```

#### TypeScript  
```typescript  
class AsyncQueue<T> {
    private queue: T[] = [];
    private maxSize: number;
    private promises: ((value: T) => void)[] = [];
    
    constructor(maxSize: number) {
        this.maxSize = maxSize;
    }
    
    async put(item: T): Promise<void> {
        if (this.queue.length >= this.maxSize) {
            return new Promise((resolve) => 
                setTimeout(() => this.put(item).then(resolve), 100)
            );
        }
        
        if (this.promises.length > 0) {
            const resolve = this.promises.shift()!;
            resolve(item);
        } else {
            this.queue.push(item);
        }
    }
    
    async take(): Promise<T> {
        if (this.queue.length > 0) {
            return this.queue.shift()!;
        }
        
        return new Promise((resolve) => {
            this.promises.push(resolve);
        });
    }
}

class ProducerConsumer {
    private queue: AsyncQueue<Task>;
    private running = false;
    
    constructor(capacity: number) {
        this.queue = new AsyncQueue(capacity);
    }
    
    async produce(task: Task): Promise<void> {
        await this.queue.put(task);
    }
    
    async startConsumer(): Promise<void> {
        this.running = true;
        while (this.running) {
            const task = await this.queue.take();
            await this.processTask(task);
        }
    }
    
    private async processTask(task: Task): Promise<void> {
        // Process the task
    }
    
    stop(): void {
        this.running = false;
    }
}
```

#### Python  
```python  
import asyncio
from asyncio import Queue
from typing import TypeVar, Generic

T = TypeVar('T')

class ProducerConsumer(Generic[T]):
    def __init__(self, capacity: int, num_consumers: int):
        self.queue: Queue[T] = Queue(capacity)
        self.num_consumers = num_consumers
        self.consumers: list[asyncio.Task] = []
        
    async def produce(self, item: T):
        await self.queue.put(item)
    
    async def consume(self):
        while True:
            try:
                item = await self.queue.get()
                await self.process_item(item)
                self.queue.task_done()
            except asyncio.CancelledError:
                break
    
    async def process_item(self, item: T):
        # Process the item
        pass
    
    async def start(self):
        self.consumers = [
            asyncio.create_task(self.consume())
            for _ in range(self.num_consumers)
        ]
    
    async def stop(self):
        for consumer in self.consumers:
            consumer.cancel()
        await asyncio.gather(*self.consumers, return_exceptions=True)

# Usage
async def main():
    pc = ProducerConsumer[str](capacity=100, num_consumers=4)
    await pc.start()
    
    # Produce items
    await pc.produce("task1")
    await pc.produce("task2")
    
    # Stop consumers
    await pc.stop()

if __name__ == "__main__":
    asyncio.run(main())
```

#### Go  
```go  
package main

import (
    "context"
    "fmt"
    "sync"
)

type Task interface{}

type ProducerConsumer struct {
    queue     chan Task
    numWorkers int
    ctx        context.Context
    cancel     context.CancelFunc
    wg         sync.WaitGroup
}

func NewProducerConsumer(capacity, numWorkers int) *ProducerConsumer {
    ctx, cancel := context.WithCancel(context.Background())
    return &ProducerConsumer{
        queue:      make(chan Task, capacity),
        numWorkers: numWorkers,
        ctx:        ctx,
        cancel:     cancel,
    }
}

func (pc *ProducerConsumer) Produce(task Task) error {
    select {
    case pc.queue <- task:
        return nil
    case <-pc.ctx.Done():
        return pc.ctx.Err()
    }
}

func (pc *ProducerConsumer) consume() {
    defer pc.wg.Done()
    
    for {
        select {
        case task := <-pc.queue:
            pc.processTask(task)
        case <-pc.ctx.Done():
            return
        }
    }
}

func (pc *ProducerConsumer) processTask(task Task) {
    // Process the task
    fmt.Printf("Processing task: %v\n", task)
}

func (pc *ProducerConsumer) Start() {
    for i := 0; i < pc.numWorkers; i++ {
        pc.wg.Add(1)
        go pc.consume()
    }
}

func (pc *ProducerConsumer) Stop() {
    pc.cancel()
    pc.wg.Wait()
}

// Usage
func main() {
    pc := NewProducerConsumer(100, 4)
    pc.Start()
    
    // Produce items
    pc.Produce("task1")
    pc.Produce("task2")
    
    // Stop and wait
    pc.Stop()
}
```

**Key Differences:**  
- Java uses BlockingQueue for thread-safe operations
- TypeScript implements async queue with Promises
- Python leverages asyncio for async operations
- Go uses channels and goroutines for concurrency

**WHEN SHOULD YOU USE IT?**  
- When processing and generation rates differ
- When you need to handle backpressure
- When multiple threads produce/consume data
- When you need to buffer or batch operations

**WHERE YOU'VE ALREADY SEEN IT**  
- Message queues (RabbitMQ, Kafka)
- Thread pools and task queues
- Event processing systems
- Streaming data processing
