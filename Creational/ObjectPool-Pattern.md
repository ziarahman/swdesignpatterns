# Design Patterns: Creational - Object Pool

## Object Pool Pattern - Simple Explanation

**SITUATION: YOU'RE CREATING EXPENSIVE OBJECTS**  
You have objects that are expensive to create, like database connections.

Initially, you have:

#### Java
```java
class DatabaseConnection {
    public DatabaseConnection() {
        // Expensive initialization
        Thread.sleep(1000);
    }
}

class Service {
    void processRequest() {
        DatabaseConnection conn = new DatabaseConnection();
        // Use connection
        conn.close();
    }
}
``` 
All good.

**THEN THE LOAD INCREASES**  
Now you have:
- Many concurrent requests
- Slow object creation
- Resource exhaustion
- Performance bottlenecks
- Memory pressure

You're stuck with:
```java
for (Request request : requests) {
    DatabaseConnection conn = new DatabaseConnection(); // Slow!
    try {
        process(conn, request);
    } finally {
        conn.close();
    }
}
``` 
Creating and destroying objects constantly!

**WHAT'S THE PROBLEM?**  
- Creating objects is expensive
- Resources are wasted
- Performance suffers
- Memory thrashing occurs
- No resource limits

**HOW OBJECT POOL SAVES YOU**  
Create a pool of reusable objects.

You say: "Give me an object from the pool." 

And the Pool:
- Manages object lifecycle
- Reuses objects
- Handles concurrent access
- Enforces limits
- Optimizes performance

Now you just:
```java
ObjectPool<DatabaseConnection> pool = new ConnectionPool();
try (DatabaseConnection conn = pool.acquire()) {
    process(conn, request);
} // Automatically returns to pool
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java
```java
public class ObjectPool<T> {
    private final ConcurrentLinkedQueue<T> pool;
    private final Semaphore semaphore;
    private final int maxSize;
    private final Supplier<T> factory;
    
    public ObjectPool(int maxSize, Supplier<T> factory) {
        this.maxSize = maxSize;
        this.factory = factory;
        this.pool = new ConcurrentLinkedQueue<>();
        this.semaphore = new Semaphore(maxSize);
    }
    
    public T acquire() throws InterruptedException {
        semaphore.acquire();
        T instance = pool.poll();
        if (instance == null) {
            instance = factory.get();
        }
        return instance;
    }
    
    public void release(T instance) {
        if (pool.offer(instance)) {
            semaphore.release();
        }
    }
}

// Usage
class ConnectionPool {
    private static final ObjectPool<DatabaseConnection> pool = 
        new ObjectPool<>(10, DatabaseConnection::new);
        
    public static DatabaseConnection acquire() {
        return pool.acquire();
    }
    
    public static void release(DatabaseConnection conn) {
        pool.release(conn);
    }
}
```

#### TypeScript
```typescript
class ObjectPool<T> {
    private pool: T[] = [];
    private inUse: Set<T> = new Set();
    
    constructor(
        private factory: () => T,
        private maxSize: number,
        private validator: (item: T) => boolean = () => true
    ) {}
    
    async acquire(): Promise<T> {
        // Try to get an available object
        let item = this.pool.find(i => !this.inUse.has(i) && this.validator(i));
        
        // Create new if none available and under max size
        if (!item && (this.pool.length < this.maxSize)) {
            item = this.factory();
            this.pool.push(item);
        }
        
        // Wait if no items available
        if (!item) {
            await new Promise(resolve => 
                setTimeout(resolve, 100)
            );
            return this.acquire();
        }
        
        this.inUse.add(item);
        return item;
    }
    
    release(item: T): void {
        this.inUse.delete(item);
    }
}

// Usage
class ConnectionPool {
    private static pool = new ObjectPool<DatabaseConnection>(
        () => new DatabaseConnection(),
        10,
        conn => conn.isValid()
    );
    
    static async acquire() {
        return await this.pool.acquire();
    }
    
    static release(conn: DatabaseConnection) {
        this.pool.release(conn);
    }
}
```

#### Python
```python
from queue import Queue
from threading import Semaphore
from typing import TypeVar, Generic, Callable
from contextlib import contextmanager

T = TypeVar('T')

class ObjectPool(Generic[T]):
    def __init__(
        self, 
        factory: Callable[[], T],
        max_size: int,
        validator: Callable[[T], bool] = lambda _: True
    ):
        self.factory = factory
        self.pool: Queue[T] = Queue()
        self.semaphore = Semaphore(max_size)
        self.validator = validator
        
    @contextmanager
    def acquire(self) -> T:
        self.semaphore.acquire()
        try:
            # Try to get from pool
            try:
                item = self.pool.get_nowait()
                while not self.validator(item):
                    item = self.factory()
            except Empty:
                item = self.factory()
                
            yield item
        finally:
            self.pool.put(item)
            self.semaphore.release()

# Usage
class ConnectionPool:
    _pool = ObjectPool(
        factory=DatabaseConnection,
        max_size=10,
        validator=lambda conn: conn.is_valid()
    )
    
    @classmethod
    @contextmanager
    def connection(cls):
        with cls._pool.acquire() as conn:
            yield conn
```

#### Go
```go
package main

import (
    "sync"
    "time"
)

type Pool[T any] struct {
    mu       sync.Mutex
    items    []T
    factory  func() T
    maxSize  int
    validate func(T) bool
}

func NewPool[T any](
    factory func() T,
    maxSize int,
    validate func(T) bool,
) *Pool[T] {
    return &Pool[T]{
        factory:  factory,
        maxSize:  maxSize,
        validate: validate,
    }
}

func (p *Pool[T]) Acquire() T {
    p.mu.Lock()
    defer p.mu.Unlock()
    
    // Find available valid item
    for i, item := range p.items {
        if p.validate(item) {
            p.items = append(p.items[:i], p.items[i+1:]...)
            return item
        }
    }
    
    // Create new if under max size
    if len(p.items) < p.maxSize {
        return p.factory()
    }
    
    // Wait and retry if pool is full
    p.mu.Unlock()
    time.Sleep(100 * time.Millisecond)
    p.mu.Lock()
    return p.Acquire()
}

func (p *Pool[T]) Release(item T) {
    p.mu.Lock()
    defer p.mu.Unlock()
    
    if p.validate(item) {
        p.items = append(p.items, item)
    }
}

// Usage
type ConnectionPool struct {
    pool *Pool[*DatabaseConnection]
}

func NewConnectionPool() *ConnectionPool {
    return &ConnectionPool{
        pool: NewPool(
            func() *DatabaseConnection { return NewDatabaseConnection() },
            10,
            func(conn *DatabaseConnection) bool { return conn.IsValid() },
        ),
    }
}
```

**Key Differences:**
- Java uses ConcurrentLinkedQueue and Semaphore
- TypeScript implements with async/await
- Python leverages context managers
- Go uses generics and mutex

**WHEN SHOULD YOU USE IT?**
- When object creation is expensive
- When you need resource limits
- When performance is critical
- When managing limited resources
- When handling concurrent access

**WHERE YOU'VE ALREADY SEEN IT**
- Database connection pools
- Thread pools
- Memory pools
- HTTP client pools
- Game object pools
