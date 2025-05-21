# Design Patterns: Concurrency - Monitor

## Monitor Pattern - Simple Explanation

**SITUATION: YOU'RE SHARING RESOURCES**  
You have a shared resource accessed by multiple threads.

Initially, you have:

#### Java
```java
class Counter {
    private int count = 0;
    
    public void increment() {
        count++;
    }
    
    public int getCount() {
        return count;
    }
}
``` 
All good.

**THEN THREADING ISSUES APPEAR**  
Now you have:
- Race conditions
- Data corruption
- Lost updates
- Inconsistent reads
- Deadlocks

You're stuck with:
```java
class Counter {
    private int count = 0;
    private final Object lock = new Object();
    
    public void increment() {
        synchronized(lock) {
            count++;
        }
    }
    
    public int getCount() {
        synchronized(lock) {
            return count;
        }
    }
    
    public void incrementIfPositive() {
        synchronized(lock) {
            if (count > 0) {
                // What if count changes here?
                count++;
            }
        }
    }
}
``` 
Locks everywhere!

**WHAT'S THE PROBLEM?**  
- Scattered synchronization
- Complex condition handling
- No structured access
- Easy to forget locks
- Deadlock prone

**HOW MONITOR SAVES YOU**  
Create a monitor object that encapsulates synchronized data with its operations.

You say: "Monitor, manage this resource safely." 

And the Monitor:
- Encapsulates data
- Manages synchronization
- Handles conditions
- Ensures atomicity
- Prevents deadlocks

Now you just:
```java
Monitor<Counter> counter = new Monitor<>();
counter.execute(c -> c.increment());
int value = counter.read(Counter::getCount);
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java
```java
public class Monitor<T> {
    private final T resource;
    private final ReentrantLock lock;
    private final Map<String, Condition> conditions;
    
    public Monitor(T resource) {
        this.resource = resource;
        this.lock = new ReentrantLock();
        this.conditions = new ConcurrentHashMap<>();
    }
    
    public void execute(Consumer<T> action) {
        lock.lock();
        try {
            action.accept(resource);
        } finally {
            lock.unlock();
        }
    }
    
    public <R> R read(Function<T, R> func) {
        lock.lock();
        try {
            return func.apply(resource);
        } finally {
            lock.unlock();
        }
    }
    
    public Condition getCondition(String name) {
        return conditions.computeIfAbsent(name, 
            k -> lock.newCondition());
    }
    
    public void await(String conditionName) throws InterruptedException {
        getCondition(conditionName).await();
    }
    
    public void signal(String conditionName) {
        getCondition(conditionName).signal();
    }
}

// Usage
class BoundedBuffer<T> {
    private final Monitor<Queue<T>> monitor;
    private final int capacity;
    
    public BoundedBuffer(int capacity) {
        this.capacity = capacity;
        this.monitor = new Monitor<>(new LinkedList<>());
    }
    
    public void put(T item) throws InterruptedException {
        monitor.execute(queue -> {
            while (queue.size() == capacity) {
                monitor.await("notFull");
            }
            queue.add(item);
            monitor.signal("notEmpty");
        });
    }
    
    public T take() throws InterruptedException {
        return monitor.read(queue -> {
            while (queue.isEmpty()) {
                monitor.await("notEmpty");
            }
            T item = queue.remove();
            monitor.signal("notFull");
            return item;
        });
    }
}
```

#### TypeScript
```typescript
class Monitor<T> {
    private resource: T;
    private mutex = new Mutex();
    private conditions = new Map<string, ConditionVariable>();
    
    constructor(resource: T) {
        this.resource = resource;
    }
    
    async execute(action: (resource: T) => Promise<void>): Promise<void> {
        await this.mutex.acquire();
        try {
            await action(this.resource);
        } finally {
            this.mutex.release();
        }
    }
    
    async read<R>(func: (resource: T) => Promise<R>): Promise<R> {
        await this.mutex.acquire();
        try {
            return await func(this.resource);
        } finally {
            this.mutex.release();
        }
    }
    
    getCondition(name: string): ConditionVariable {
        let condition = this.conditions.get(name);
        if (!condition) {
            condition = new ConditionVariable(this.mutex);
            this.conditions.set(name, condition);
        }
        return condition;
    }
}

// Usage
class BoundedBuffer<T> {
    private monitor: Monitor<T[]>;
    private capacity: number;
    
    constructor(capacity: number) {
        this.capacity = capacity;
        this.monitor = new Monitor<T[]>([]);
    }
    
    async put(item: T): Promise<void> {
        await this.monitor.execute(async (buffer) => {
            while (buffer.length === this.capacity) {
                await this.monitor.getCondition('notFull').wait();
            }
            buffer.push(item);
            this.monitor.getCondition('notEmpty').signal();
        });
    }
    
    async take(): Promise<T> {
        return await this.monitor.read(async (buffer) => {
            while (buffer.length === 0) {
                await this.monitor.getCondition('notEmpty').wait();
            }
            const item = buffer.shift()!;
            this.monitor.getCondition('notFull').signal();
            return item;
        });
    }
}
```

#### Python
```python
from threading import Lock, Condition
from typing import TypeVar, Generic, Callable, Any
from collections import deque

T = TypeVar('T')
R = TypeVar('R')

class Monitor(Generic[T]):
    def __init__(self, resource: T):
        self.resource = resource
        self.lock = Lock()
        self.conditions = {}
    
    def execute(self, action: Callable[[T], None]) -> None:
        with self.lock:
            action(self.resource)
    
    def read(self, func: Callable[[T], R]) -> R:
        with self.lock:
            return func(self.resource)
    
    def get_condition(self, name: str) -> Condition:
        if name not in self.conditions:
            self.conditions[name] = Condition(self.lock)
        return self.conditions[name]
    
    def __enter__(self):
        self.lock.acquire()
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.lock.release()

# Usage
class BoundedBuffer(Generic[T]):
    def __init__(self, capacity: int):
        self.monitor = Monitor(deque(maxlen=capacity))
        self.capacity = capacity
    
    def put(self, item: T) -> None:
        with self.monitor as m:
            while len(m.resource) == self.capacity:
                m.get_condition('not_full').wait()
            m.resource.append(item)
            m.get_condition('not_empty').notify()
    
    def take(self) -> T:
        with self.monitor as m:
            while len(m.resource) == 0:
                m.get_condition('not_empty').wait()
            item = m.resource.popleft()
            m.get_condition('not_full').notify()
            return item
```

#### Go
```go
package main

import (
    "sync"
)

type Monitor[T any] struct {
    resource T
    mutex    sync.RWMutex
    conds    map[string]*sync.Cond
}

func NewMonitor[T any](resource T) *Monitor[T] {
    m := &Monitor[T]{
        resource: resource,
        conds:    make(map[string]*sync.Cond),
    }
    return m
}

func (m *Monitor[T]) Execute(action func(T)) {
    m.mutex.Lock()
    defer m.mutex.Unlock()
    action(m.resource)
}

func (m *Monitor[T]) Read(action func(T) interface{}) interface{} {
    m.mutex.RLock()
    defer m.mutex.RUnlock()
    return action(m.resource)
}

func (m *Monitor[T]) GetCondition(name string) *sync.Cond {
    if cond, exists := m.conds[name]; exists {
        return cond
    }
    cond := sync.NewCond(&m.mutex)
    m.conds[name] = cond
    return cond
}

// Usage
type BoundedBuffer[T any] struct {
    monitor  *Monitor[[]T]
    capacity int
}

func NewBoundedBuffer[T any](capacity int) *BoundedBuffer[T] {
    return &BoundedBuffer[T]{
        monitor:  NewMonitor(make([]T, 0, capacity)),
        capacity: capacity,
    }
}

func (bb *BoundedBuffer[T]) Put(item T) {
    bb.monitor.Execute(func(buffer []T) {
        for len(buffer) == bb.capacity {
            bb.monitor.GetCondition("notFull").Wait()
        }
        buffer = append(buffer, item)
        bb.monitor.GetCondition("notEmpty").Signal()
    })
}

func (bb *BoundedBuffer[T]) Take() T {
    var result T
    bb.monitor.Execute(func(buffer []T) {
        for len(buffer) == 0 {
            bb.monitor.GetCondition("notEmpty").Wait()
        }
        result = buffer[0]
        buffer = buffer[1:]
        bb.monitor.GetCondition("notFull").Signal()
    })
    return result
}
```

**Key Differences:**
- Java uses ReentrantLock and Conditions
- TypeScript implements with async/await
- Python uses context managers
- Go leverages sync.Mutex and sync.Cond

**WHEN SHOULD YOU USE IT?**
- When sharing resources between threads
- When synchronization is complex
- When handling condition variables
- When atomic operations are needed
- When thread safety is critical

**WHERE YOU'VE ALREADY SEEN IT**
- Database connection pools
- Thread-safe collections
- Resource managers
- Buffered channels
- Synchronized queues
