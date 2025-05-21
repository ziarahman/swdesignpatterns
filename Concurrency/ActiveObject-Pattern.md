# Design Patterns: Concurrency - Active Object

## Active Object Pattern - Simple Explanation

**SITUATION: YOU'RE MANAGING ASYNC OPERATIONS**  
You have an object that needs to handle requests asynchronously. 

Initially, you have: 

#### Java
```java
class Service {
    public void doTask() {
        // Do work synchronously
    }
}
``` 
All good.

**THEN THE COMPLEXITY GROWS**  
Now you need:  
- Async operations
- Request queuing
- Result handling
- Thread safety

You're stuck with: 
```java
class Service {
    public Future<Result> doTask() {
        return executorService.submit(() -> {
            // Complex thread handling
            // Manual synchronization
            // Error-prone concurrency
        });
    }
}
``` 
Getting messy!

**WHAT'S THE PROBLEM?**  
- Mixed concerns (business logic and threading)
- Complex thread management
- Hard to handle request ordering
- Difficult error handling

**HOW ACTIVE OBJECT SAVES YOU**  
Separate method execution from method invocation.

You say: "Handle this request asynchronously but make it look synchronous." 

And the Pattern:  
- Decouples method invocation from execution
- Manages its own thread of control
- Queues incoming requests
- Returns futures for results

Now you just: 
```java
ActiveService service = new ActiveService();
Future<Result> future = service.doTask();
Result result = future.get(); // When you need it
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
// The servant class that does the actual work
class Servant {
    public String doTask(String param) {
        // Simulate work
        return "Processed: " + param;
    }
}

// Active object proxy
class ActiveObject {
    private final BlockingQueue<Runnable> queue;
    private final Servant servant;
    private final ExecutorService executor;
    
    public ActiveObject() {
        this.queue = new LinkedBlockingQueue<>();
        this.servant = new Servant();
        this.executor = Executors.newSingleThreadExecutor();
        start();
    }
    
    public Future<String> doTask(String param) {
        CompletableFuture<String> future = new CompletableFuture<>();
        queue.offer(() -> {
            try {
                String result = servant.doTask(param);
                future.complete(result);
            } catch (Exception e) {
                future.completeExceptionally(e);
            }
        });
        return future;
    }
    
    private void start() {
        executor.execute(() -> {
            while (!Thread.currentThread().isInterrupted()) {
                try {
                    Runnable task = queue.take();
                    task.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
        });
    }
    
    public void shutdown() {
        executor.shutdown();
    }
}
```

#### TypeScript  
```typescript  
class Servant {
    doTask(param: string): Promise<string> {
        return new Promise(resolve => {
            // Simulate work
            setTimeout(() => {
                resolve(`Processed: ${param}`);
            }, 100);
        });
    }
}

class ActiveObject {
    private servant: Servant;
    private queue: Array<() => Promise<void>>;
    private isProcessing: boolean;
    
    constructor() {
        this.servant = new Servant();
        this.queue = [];
        this.isProcessing = false;
    }
    
    async doTask(param: string): Promise<string> {
        return new Promise((resolve, reject) => {
            this.queue.push(async () => {
                try {
                    const result = await this.servant.doTask(param);
                    resolve(result);
                } catch (error) {
                    reject(error);
                }
            });
            
            this.processQueue();
        });
    }
    
    private async processQueue(): Promise<void> {
        if (this.isProcessing) return;
        
        this.isProcessing = true;
        while (this.queue.length > 0) {
            const task = this.queue.shift()!;
            await task();
        }
        this.isProcessing = false;
    }
}
```

#### Python  
```python  
import asyncio
from typing import TypeVar, Generic, Callable, Awaitable
from collections import deque

T = TypeVar('T')
R = TypeVar('R')

class Servant:
    async def do_task(self, param: str) -> str:
        # Simulate work
        await asyncio.sleep(0.1)
        return f"Processed: {param}"

class ActiveObject:
    def __init__(self):
        self.servant = Servant()
        self.queue: deque[Callable[[], Awaitable[None]]] = deque()
        self.processing = False
        self.lock = asyncio.Lock()
    
    async def do_task(self, param: str) -> str:
        future: asyncio.Future[str] = asyncio.Future()
        
        async def task():
            try:
                result = await self.servant.do_task(param)
                future.set_result(result)
            except Exception as e:
                future.set_exception(e)
        
        self.queue.append(task)
        await self._process_queue()
        return await future
    
    async def _process_queue(self):
        if self.processing:
            return
        
        async with self.lock:
            self.processing = True
            while self.queue:
                task = self.queue.popleft()
                await task()
            self.processing = False

# Usage
async def main():
    active = ActiveObject()
    result = await active.do_task("test")
    print(result)

if __name__ == "__main__":
    asyncio.run(main())
```

#### Go  
```go  
package main

import (
    "fmt"
    "time"
)

type Request struct {
    param     string
    response  chan string
}

type Servant struct{}

func (s *Servant) doTask(param string) string {
    // Simulate work
    time.Sleep(100 * time.Millisecond)
    return fmt.Sprintf("Processed: %s", param)
}

type ActiveObject struct {
    requests chan Request
    quit     chan struct{}
}

func NewActiveObject() *ActiveObject {
    ao := &ActiveObject{
        requests: make(chan Request),
        quit:     make(chan struct{}),
    }
    go ao.serve()
    return ao
}

func (ao *ActiveObject) serve() {
    servant := &Servant{}
    
    for {
        select {
        case req := <-ao.requests:
            result := servant.doTask(req.param)
            req.response <- result
        case <-ao.quit:
            return
        }
    }
}

func (ao *ActiveObject) DoTask(param string) <-chan string {
    response := make(chan string, 1)
    ao.requests <- Request{param: param, response: response}
    return response
}

func (ao *ActiveObject) Stop() {
    close(ao.quit)
}

// Usage
func main() {
    ao := NewActiveObject()
    defer ao.Stop()
    
    result := <-ao.DoTask("test")
    fmt.Println(result)
}
```

**Key Differences:**  
- Java uses ExecutorService and CompletableFuture
- TypeScript implements async queue with Promises
- Python leverages asyncio for asynchronous processing
- Go uses channels for communication and synchronization

**WHEN SHOULD YOU USE IT?**  
- When you need thread-safe object access
- When method execution should be asynchronous
- When you need to queue and order requests
- When you want to hide threading complexity

**WHERE YOU'VE ALREADY SEEN IT**  
- GUI event dispatchers
- Network service handlers
- Game engine components
- Async service interfaces
