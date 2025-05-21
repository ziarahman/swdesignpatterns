# Design Patterns: Concurrency - Barrier

## Barrier Pattern - Simple Explanation

**SITUATION: YOU'RE COORDINATING PARALLEL WORK**  
You have multiple threads that need to wait for each other at certain points.

Initially, you have:

#### Java
```java
class DataProcessor {
    void processPartition(List<Data> partition) {
        process(partition);
        // Continue with next phase
    }
}
``` 
All good.

**THEN THE DEPENDENCIES APPEAR**  
Now you need:
- Phase synchronization
- All threads to complete
- Coordinated restarts
- Progress tracking
- Error handling

You're stuck with:
```java
class DataProcessor {
    static volatile int completed = 0;
    static final Object lock = new Object();
    
    void processPartition(List<Data> partition, int totalThreads) {
        process(partition);
        
        synchronized(lock) {
            completed++;
            if (completed == totalThreads) {
                // Ready for next phase
                completed = 0;
                lock.notifyAll();
            } else {
                while (completed < totalThreads) {
                    lock.wait();
                }
            }
        }
    }
}
``` 
Synchronization spaghetti!

**WHAT'S THE PROBLEM?**  
- Complex coordination logic
- Error-prone synchronization
- Hard to manage threads
- Difficult error handling
- No reusable solution

**HOW BARRIER SAVES YOU**  
Create a barrier that threads must wait at until everyone arrives.

You say: "Wait at this point until all threads are ready." 

And the Barrier:
- Coordinates threads
- Manages synchronization
- Tracks progress
- Handles timeouts
- Provides clean restarts

Now you just:
```java
Barrier barrier = new Barrier(threadCount);
// In each thread:
processPhaseOne();
barrier.await(); // Wait for all threads
processPhaseTwo();
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java
```java
public class Barrier {
    private final int parties;
    private int count;
    private final Runnable barrierAction;
    private Generation generation;
    
    private static class Generation {
        boolean broken;
    }
    
    public Barrier(int parties, Runnable barrierAction) {
        this.parties = parties;
        this.count = parties;
        this.barrierAction = barrierAction;
        this.generation = new Generation();
    }
    
    public Barrier(int parties) {
        this(parties, null);
    }
    
    public synchronized void await() throws InterruptedException {
        Generation g = generation;
        if (g.broken) {
            throw new BrokenBarrierException();
        }
        
        int index = --count;
        if (index == 0) {  // Last thread to arrive
            if (barrierAction != null) {
                barrierAction.run();
            }
            nextGeneration();
            return;
        }
        
        // Wait until all threads arrive or barrier is broken
        while (count > 0 && !g.broken) {
            try {
                wait();
            } catch (InterruptedException e) {
                breakBarrier();
                throw e;
            }
        }
        
        if (g.broken) {
            throw new BrokenBarrierException();
        }
    }
    
    private void nextGeneration() {
        // Wake up waiting threads
        generation = new Generation();
        count = parties;
        notifyAll();
    }
    
    private void breakBarrier() {
        generation.broken = true;
        count = parties;
        notifyAll();
    }
}

// Usage
class ParallelProcessor {
    private final int threadCount;
    private final Barrier barrier;
    
    public ParallelProcessor(int threadCount) {
        this.threadCount = threadCount;
        this.barrier = new Barrier(threadCount);
    }
    
    public void process(List<List<Data>> partitions) {
        List<Thread> threads = new ArrayList<>();
        
        for (int i = 0; i < threadCount; i++) {
            final int index = i;
            Thread t = new Thread(() -> {
                try {
                    processPhaseOne(partitions.get(index));
                    barrier.await();
                    processPhaseTwo(partitions.get(index));
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
            threads.add(t);
            t.start();
        }
        
        threads.forEach(t -> {
            try {
                t.join();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
    }
}
```

#### TypeScript
```typescript
class Barrier {
    private count: number;
    private parties: number;
    private generation: number = 0;
    private callbacks: (() => void)[] = [];
    
    constructor(parties: number) {
        this.parties = parties;
        this.count = parties;
    }
    
    async await(): Promise<void> {
        const currentGeneration = this.generation;
        const myCallback = new Promise<void>((resolve) => {
            this.callbacks.push(resolve);
        });
        
        this.count--;
        
        if (this.count === 0) {
            // Last thread arrived
            this.count = this.parties;
            this.generation++;
            this.callbacks.forEach(cb => cb());
            this.callbacks = [];
            return;
        }
        
        // Wait for this generation to complete
        await myCallback;
        
        // Verify we're still in the same generation
        if (currentGeneration !== this.generation - 1) {
            throw new Error('Barrier was reset');
        }
    }
    
    reset(): void {
        this.count = this.parties;
        this.generation++;
        this.callbacks.forEach(cb => cb());
        this.callbacks = [];
    }
}

// Usage
class ParallelProcessor {
    private barrier: Barrier;
    
    constructor(private threadCount: number) {
        this.barrier = new Barrier(threadCount);
    }
    
    async process(partitions: Data[][]): Promise<void> {
        const tasks = partitions.map(async (partition, i) => {
            await this.processPhaseOne(partition);
            await this.barrier.await();
            await this.processPhaseTwo(partition);
        });
        
        await Promise.all(tasks);
    }
}
```

#### Python
```python
from threading import Condition, BrokenBarrierError
from typing import Optional, Callable

class Barrier:
    def __init__(self, parties: int, action: Optional[Callable[[], None]] = None):
        self.parties = parties
        self.count = parties
        self.action = action
        self.generation = 0
        self.broken = False
        self.condition = Condition()
    
    def wait(self) -> int:
        with self.condition:
            generation = self.generation
            
            if self.broken:
                raise BrokenBarrierError()
            
            self.count -= 1
            if self.count == 0:
                # Last thread to arrive
                if self.action:
                    self.action()
                self.generation += 1
                self.count = self.parties
                self.condition.notify_all()
                return 0
            
            # Wait for all threads
            while (self.count > 0 and 
                   self.generation == generation and 
                   not self.broken):
                self.condition.wait()
            
            if self.broken:
                raise BrokenBarrierError()
            
            return self.count

    def reset(self) -> None:
        with self.condition:
            self.broken = False
            self.generation += 1
            self.count = self.parties
            self.condition.notify_all()
    
    def break_barrier(self) -> None:
        with self.condition:
            self.broken = True
            self.generation += 1
            self.count = self.parties
            self.condition.notify_all()

# Usage
from threading import Thread
from typing import List

class ParallelProcessor:
    def __init__(self, thread_count: int):
        self.thread_count = thread_count
        self.barrier = Barrier(thread_count)
    
    def process(self, partitions: List[List[Data]]):
        threads = []
        
        def worker(partition: List[Data]):
            try:
                self.process_phase_one(partition)
                self.barrier.wait()
                self.process_phase_two(partition)
            except BrokenBarrierError:
                print("Barrier was broken")
        
        for i in range(self.thread_count):
            t = Thread(target=worker, args=(partitions[i],))
            threads.append(t)
            t.start()
        
        for t in threads:
            t.join()
```

#### Go
```go
package main

import (
    "context"
    "sync"
)

type Barrier struct {
    parties    int
    count      int
    generation int
    lock       sync.Mutex
    cond       *sync.Cond
}

func NewBarrier(parties int) *Barrier {
    b := &Barrier{
        parties: parties,
        count:   parties,
    }
    b.cond = sync.NewCond(&b.lock)
    return b
}

func (b *Barrier) Await(ctx context.Context) error {
    b.lock.Lock()
    defer b.lock.Unlock()
    
    generation := b.generation
    
    b.count--
    if b.count == 0 {
        // Last goroutine to arrive
        b.count = b.parties
        b.generation++
        b.cond.Broadcast()
        return nil
    }
    
    // Wait for all goroutines
    for b.count > 0 && b.generation == generation {
        done := make(chan struct{})
        go func() {
            b.cond.Wait()
            close(done)
        }()
        
        select {
        case <-done:
        case <-ctx.Done():
            b.count++
            if b.count == b.parties {
                b.generation++
            }
            b.cond.Broadcast()
            return ctx.Err()
        }
    }
    
    return nil
}

// Usage
type ParallelProcessor struct {
    threadCount int
    barrier     *Barrier
}

func NewParallelProcessor(threadCount int) *ParallelProcessor {
    return &ParallelProcessor{
        threadCount: threadCount,
        barrier:     NewBarrier(threadCount),
    }
}

func (p *ParallelProcessor) Process(ctx context.Context, partitions [][]Data) error {
    var wg sync.WaitGroup
    errChan := make(chan error, p.threadCount)
    
    for i := 0; i < p.threadCount; i++ {
        wg.Add(1)
        go func(index int) {
            defer wg.Done()
            
            if err := p.processPhaseOne(partitions[index]); err != nil {
                errChan <- err
                return
            }
            
            if err := p.barrier.Await(ctx); err != nil {
                errChan <- err
                return
            }
            
            if err := p.processPhaseTwo(partitions[index]); err != nil {
                errChan <- err
                return
            }
        }(i)
    }
    
    go func() {
        wg.Wait()
        close(errChan)
    }()
    
    for err := range errChan {
        if err != nil {
            return err
        }
    }
    
    return nil
}
```

**Key Differences:**
- Java uses synchronized blocks and wait/notify
- TypeScript implements with async/await and Promises
- Python uses Condition variables
- Go leverages channels and context

**WHEN SHOULD YOU USE IT?**
- When synchronizing parallel phases
- When coordinating multiple threads
- When implementing phased computations
- When managing parallel algorithms
- When ensuring group progress

**WHERE YOU'VE ALREADY SEEN IT**
- Parallel algorithms
- Scientific computing
- Data processing pipelines
- Game physics engines
- Distributed systems
