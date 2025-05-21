# Bulkhead Pattern

## Simple Explanation
Think of a ship's bulkhead compartments - if one compartment floods, the others remain safe, keeping the ship afloat. Similarly, the Bulkhead pattern isolates parts of your application into pools or compartments, so if one fails, the others continue to function.

## Real-World Scenario
Imagine an e-commerce platform with:
- Product catalog browsing
- Shopping cart management
- Payment processing
- Order fulfillment

Without Bulkheads:
- A surge in catalog searches could overwhelm the entire system
- Payment processing issues could crash the whole application
- One slow service affects all users

With Bulkheads:
- Each service has dedicated resources
- Failures are contained
- Critical services remain available

## Problem Statement
In distributed systems, we face:
- Resource exhaustion
- Cascading failures
- Uneven load distribution
- Service dependencies
- System-wide outages

## Solution
The Bulkhead pattern provides:
1. Resource Isolation
   - Separate thread pools
   - Dedicated connection pools
   - Isolated service instances

2. Failure Containment
   - Limited blast radius
   - Protected critical services
   - Controlled degradation

3. Resource Management
   - Pool sizing
   - Queue management
   - Load balancing

## Implementation Examples

### TypeScript Implementation
```typescript
class BulkheadPool {
    private readonly maxConcurrent: number;
    private readonly queue: Array<() => Promise<any>>;
    private currentExecuting: number;

    constructor(maxConcurrent: number, queueSize: number = 10) {
        this.maxConcurrent = maxConcurrent;
        this.queue = [];
        this.currentExecuting = 0;
    }

    async execute<T>(task: () => Promise<T>): Promise<T> {
        if (this.currentExecuting >= this.maxConcurrent) {
            if (this.queue.length >= this.queue.length) {
                throw new Error('Bulkhead queue is full');
            }
            return new Promise((resolve, reject) => {
                this.queue.push(async () => {
                    try {
                        const result = await this.executeTask(task);
                        resolve(result);
                    } catch (error) {
                        reject(error);
                    }
                });
            });
        }

        return this.executeTask(task);
    }

    private async executeTask<T>(task: () => Promise<T>): Promise<T> {
        this.currentExecuting++;
        try {
            const result = await task();
            return result;
        } finally {
            this.currentExecuting--;
            this.processQueue();
        }
    }

    private processQueue(): void {
        if (this.queue.length > 0 && this.currentExecuting < this.maxConcurrent) {
            const nextTask = this.queue.shift();
            if (nextTask) {
                nextTask();
            }
        }
    }
}

// Usage example
class ServiceClient {
    private readonly bulkhead: BulkheadPool;

    constructor(maxConcurrent: number) {
        this.bulkhead = new BulkheadPool(maxConcurrent);
    }

    async makeRequest(url: string): Promise<Response> {
        return this.bulkhead.execute(async () => {
            const response = await fetch(url);
            return response;
        });
    }
}
```

### Java Implementation
```java
public class Bulkhead {
    private final Semaphore semaphore;
    private final BlockingQueue<Runnable> queue;
    private final ExecutorService executor;

    public Bulkhead(int maxConcurrent, int queueSize) {
        this.semaphore = new Semaphore(maxConcurrent);
        this.queue = new ArrayBlockingQueue<>(queueSize);
        this.executor = Executors.newCachedThreadPool();
    }

    public <T> CompletableFuture<T> execute(Supplier<T> task) {
        CompletableFuture<T> future = new CompletableFuture<>();

        if (!semaphore.tryAcquire()) {
            if (!queue.offer(() -> executeTask(task, future))) {
                future.completeExceptionally(
                    new BulkheadFullException("Queue is full"));
            }
            return future;
        }

        executeTask(task, future);
        return future;
    }

    private <T> void executeTask(Supplier<T> task, CompletableFuture<T> future) {
        executor.submit(() -> {
            try {
                T result = task.get();
                future.complete(result);
            } catch (Exception e) {
                future.completeExceptionally(e);
            } finally {
                semaphore.release();
                processQueue();
            }
        });
    }

    private void processQueue() {
        Runnable nextTask = queue.poll();
        if (nextTask != null) {
            executor.submit(nextTask);
        }
    }

    public void shutdown() {
        executor.shutdown();
    }
}

// Usage example
class ServiceClient {
    private final Bulkhead bulkhead;

    public ServiceClient(int maxConcurrent) {
        this.bulkhead = new Bulkhead(maxConcurrent, 20);
    }

    public CompletableFuture<String> makeRequest(String url) {
        return bulkhead.execute(() -> {
            HttpClient client = HttpClient.newHttpClient();
            HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(url))
                .build();
            return client.send(request, BodyHandlers.ofString())
                .body();
        });
    }
}
```

### Python Implementation
```python
import asyncio
from dataclasses import dataclass
from typing import TypeVar, Callable, Awaitable, Optional
from collections import deque

T = TypeVar('T')

@dataclass
class BulkheadConfig:
    max_concurrent: int
    max_queue_size: int = 10

class Bulkhead:
    def __init__(self, config: BulkheadConfig):
        self.config = config
        self.semaphore = asyncio.Semaphore(config.max_concurrent)
        self.queue: deque = deque(maxlen=config.max_queue_size)
        self.current_executing = 0

    async def execute(self, task: Callable[[], Awaitable[T]]) -> T:
        if self.current_executing >= self.config.max_concurrent:
            if len(self.queue) >= self.config.max_queue_size:
                raise Exception("Bulkhead queue is full")
            
            future = asyncio.Future()
            self.queue.append((task, future))
            return await future

        return await self._execute_task(task)

    async def _execute_task(self, task: Callable[[], Awaitable[T]]) -> T:
        async with self.semaphore:
            self.current_executing += 1
            try:
                return await task()
            finally:
                self.current_executing -= 1
                await self._process_queue()

    async def _process_queue(self):
        if self.queue and self.current_executing < self.config.max_concurrent:
            task, future = self.queue.popleft()
            try:
                result = await self._execute_task(task)
                future.set_result(result)
            except Exception as e:
                future.set_exception(e)

class ServiceClient:
    def __init__(self, max_concurrent: int):
        self.bulkhead = Bulkhead(BulkheadConfig(max_concurrent))

    async def make_request(self, url: str) -> str:
        async def request():
            async with aiohttp.ClientSession() as session:
                async with session.get(url) as response:
                    return await response.text()

        return await self.bulkhead.execute(request)
```

## Usage Guidelines

1. **When to Use**
   - High-load systems
   - Critical service protection
   - Resource-intensive operations
   - Multi-tenant applications

2. **Configuration Best Practices**
   - Size pools appropriately
   - Set reasonable queue limits
   - Monitor resource usage
   - Configure timeouts
   - Implement fallbacks

3. **Implementation Considerations**
   - Thread pool sizing
   - Queue capacity
   - Timeout values
   - Monitoring metrics
   - Failure handling

## Real-World Examples

1. **Netflix Hystrix**
   - Thread pool isolation
   - Command patterns
   - Circuit breaker integration
   - Metrics and monitoring

2. **Microsoft Azure**
   - Service Fabric isolations
   - Resource governance
   - Application isolation
   - Container orchestration

3. **Amazon ECS**
   - Container isolation
   - Resource allocation
   - Task definitions
   - Service limits

## Key Implementation Differences

1. **TypeScript vs Java**
   - TypeScript uses async/await
   - Java uses CompletableFuture
   - Both implement queue management

2. **Python vs Others**
   - Python uses asyncio
   - Coroutine-based concurrency
   - Async context managers

3. **Resource Management**
   - TypeScript: Simple counters
   - Java: Semaphores and ExecutorService
   - Python: Asyncio primitives
