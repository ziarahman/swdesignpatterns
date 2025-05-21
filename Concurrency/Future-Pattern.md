# Design Patterns: Concurrency - Future

## Future Pattern - Simple Explanation

**SITUATION: YOU'RE HANDLING ASYNC OPERATIONS**  
You have long-running operations that shouldn't block.

Initially, you have:

#### Java
```java
class ImageProcessor {
    Image processImage(byte[] data) {
        // Long-running processing
        return processedImage;
    }
}

class Service {
    void handleRequest() {
        Image result = processor.processImage(data);
        // Wait for processing...
        display(result);
    }
}
``` 
All good.

**THEN THE REQUIREMENTS CHANGE**  
Now you need:
- Non-blocking operations
- Progress tracking
- Cancellation support
- Multiple async tasks
- Error handling

You're stuck with:
```java
class Service {
    void handleRequest() {
        Thread thread = new Thread(() -> {
            try {
                Image result = processor.processImage(data);
                // How to get result back?
                // How to handle errors?
                // How to cancel?
            } catch (Exception e) {
                // Error handling?
            }
        });
        thread.start();
        // How to wait for completion?
    }
}
``` 
Threading chaos!

**WHAT'S THE PROBLEM?**  
- No way to get results
- Hard to handle errors
- Can't track progress
- No cancellation support
- Complex thread management

**HOW FUTURE SAVES YOU**  
Create a placeholder for a result that will exist in the future.

You say: "Start this work and give me a Future to track it." 

And the Future:
- Represents async result
- Handles completion
- Manages errors
- Supports cancellation
- Enables composition

Now you just:
```java
Future<Image> future = executor.submit(() -> 
    processor.processImage(data)
);

// Do other work while processing...
Image result = future.get(); // Get result when needed
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java
```java
public interface Future<T> {
    boolean isDone();
    boolean isCancelled();
    boolean cancel(boolean mayInterruptIfRunning);
    T get() throws InterruptedException, ExecutionException;
    T get(long timeout, TimeUnit unit) 
        throws InterruptedException, ExecutionException, TimeoutException;
}

public class CompletableFuture<T> implements Future<T> {
    private T result;
    private Exception exception;
    private boolean completed;
    private boolean cancelled;
    private List<Consumer<T>> successCallbacks = new ArrayList<>();
    private List<Consumer<Exception>> errorCallbacks = new ArrayList<>();
    
    public synchronized void complete(T value) {
        if (!completed && !cancelled) {
            result = value;
            completed = true;
            successCallbacks.forEach(callback -> callback.accept(value));
        }
    }
    
    public synchronized void completeExceptionally(Exception ex) {
        if (!completed && !cancelled) {
            exception = ex;
            completed = true;
            errorCallbacks.forEach(callback -> callback.accept(ex));
        }
    }
    
    @Override
    public synchronized T get() throws InterruptedException, ExecutionException {
        while (!completed && !cancelled) {
            wait();
        }
        
        if (cancelled) {
            throw new CancellationException();
        }
        
        if (exception != null) {
            throw new ExecutionException(exception);
        }
        
        return result;
    }
    
    public CompletableFuture<T> thenAccept(Consumer<T> action) {
        synchronized (this) {
            if (completed && !cancelled && exception == null) {
                action.accept(result);
            } else {
                successCallbacks.add(action);
            }
        }
        return this;
    }
    
    public CompletableFuture<T> exceptionally(Consumer<Exception> handler) {
        synchronized (this) {
            if (completed && exception != null) {
                handler.accept(exception);
            } else {
                errorCallbacks.add(handler);
            }
        }
        return this;
    }
}

// Usage
class ImageService {
    private final ExecutorService executor = Executors.newCachedThreadPool();
    
    public CompletableFuture<Image> processImageAsync(byte[] data) {
        CompletableFuture<Image> future = new CompletableFuture<>();
        
        executor.submit(() -> {
            try {
                Image result = processImage(data);
                future.complete(result);
            } catch (Exception e) {
                future.completeExceptionally(e);
            }
        });
        
        return future;
    }
}
```

#### TypeScript
```typescript
interface Future<T> {
    isDone(): boolean;
    isCancelled(): boolean;
    cancel(): boolean;
    get(): Promise<T>;
    then<R>(fn: (value: T) => R | Promise<R>): Future<R>;
    catch(fn: (error: Error) => void): Future<T>;
}

class CompletableFuture<T> implements Future<T> {
    private promise: Promise<T>;
    private resolvePromise!: (value: T) => void;
    private rejectPromise!: (error: Error) => void;
    private completed = false;
    private cancelled = false;
    
    constructor() {
        this.promise = new Promise<T>((resolve, reject) => {
            this.resolvePromise = resolve;
            this.rejectPromise = reject;
        });
    }
    
    complete(value: T): void {
        if (!this.completed && !this.cancelled) {
            this.completed = true;
            this.resolvePromise(value);
        }
    }
    
    completeExceptionally(error: Error): void {
        if (!this.completed && !this.cancelled) {
            this.completed = true;
            this.rejectPromise(error);
        }
    }
    
    isDone(): boolean {
        return this.completed;
    }
    
    isCancelled(): boolean {
        return this.cancelled;
    }
    
    cancel(): boolean {
        if (!this.completed) {
            this.cancelled = true;
            this.rejectPromise(new Error('Cancelled'));
            return true;
        }
        return false;
    }
    
    async get(): Promise<T> {
        return this.promise;
    }
    
    then<R>(fn: (value: T) => R | Promise<R>): Future<R> {
        const future = new CompletableFuture<R>();
        
        this.promise
            .then(value => Promise.resolve(fn(value)))
            .then(result => future.complete(result))
            .catch(error => future.completeExceptionally(error));
        
        return future;
    }
    
    catch(fn: (error: Error) => void): Future<T> {
        this.promise.catch(fn);
        return this;
    }
}

// Usage
class ImageService {
    async processImageAsync(data: Uint8Array): Future<Image> {
        const future = new CompletableFuture<Image>();
        
        setTimeout(async () => {
            try {
                const result = await this.processImage(data);
                future.complete(result);
            } catch (error) {
                future.completeExceptionally(error);
            }
        }, 0);
        
        return future;
    }
}
```

#### Python
```python
from concurrent.futures import Future as ConcurrentFuture
from typing import TypeVar, Generic, Callable, Any
from threading import Lock
import asyncio

T = TypeVar('T')

class Future(Generic[T]):
    def __init__(self):
        self._result: T = None
        self._exception: Exception = None
        self._completed = False
        self._cancelled = False
        self._lock = Lock()
        self._callbacks = []
    
    def complete(self, result: T) -> None:
        with self._lock:
            if not self._completed and not self._cancelled:
                self._result = result
                self._completed = True
                self._notify_callbacks()
    
    def complete_exceptionally(self, exception: Exception) -> None:
        with self._lock:
            if not self._completed and not self._cancelled:
                self._exception = exception
                self._completed = True
                self._notify_callbacks()
    
    def _notify_callbacks(self) -> None:
        for callback in self._callbacks:
            try:
                callback(self)
            except Exception:
                pass
    
    def add_callback(self, fn: Callable[['Future[T]'], None]) -> None:
        with self._lock:
            if self._completed:
                fn(self)
            else:
                self._callbacks.append(fn)
    
    def cancel(self) -> bool:
        with self._lock:
            if not self._completed:
                self._cancelled = True
                self._completed = True
                return True
            return False
    
    def is_done(self) -> bool:
        return self._completed
    
    def is_cancelled(self) -> bool:
        return self._cancelled
    
    def get(self, timeout: float = None) -> T:
        if self._cancelled:
            raise CancellationError()
        if self._exception:
            raise self._exception
        if not self._completed:
            raise TimeoutError()
        return self._result

# Usage with asyncio
class ImageService:
    def process_image_async(self, data: bytes) -> Future[Image]:
        future = Future[Image]()
        
        async def process():
            try:
                result = await self.process_image(data)
                future.complete(result)
            except Exception as e:
                future.complete_exceptionally(e)
        
        asyncio.create_task(process())
        return future
```

#### Go
```go
package main

import (
    "context"
    "sync"
    "time"
)

type Future[T any] struct {
    result     T
    err        error
    done       chan struct{}
    cancelled  bool
    completed  bool
    mutex      sync.RWMutex
}

func NewFuture[T any]() *Future[T] {
    return &Future[T]{
        done: make(chan struct{}),
    }
}

func (f *Future[T]) Complete(value T) bool {
    f.mutex.Lock()
    defer f.mutex.Unlock()
    
    if !f.completed && !f.cancelled {
        f.result = value
        f.completed = true
        close(f.done)
        return true
    }
    return false
}

func (f *Future[T]) CompleteExceptionally(err error) bool {
    f.mutex.Lock()
    defer f.mutex.Unlock()
    
    if !f.completed && !f.cancelled {
        f.err = err
        f.completed = true
        close(f.done)
        return true
    }
    return false
}

func (f *Future[T]) Cancel() bool {
    f.mutex.Lock()
    defer f.mutex.Unlock()
    
    if !f.completed {
        f.cancelled = true
        f.completed = true
        close(f.done)
        return true
    }
    return false
}

func (f *Future[T]) IsDone() bool {
    f.mutex.RLock()
    defer f.mutex.RUnlock()
    return f.completed
}

func (f *Future[T]) IsCancelled() bool {
    f.mutex.RLock()
    defer f.mutex.RUnlock()
    return f.cancelled
}

func (f *Future[T]) Get(ctx context.Context) (T, error) {
    select {
    case <-f.done:
        f.mutex.RLock()
        defer f.mutex.RUnlock()
        
        if f.cancelled {
            var zero T
            return zero, ErrCancelled
        }
        
        if f.err != nil {
            var zero T
            return zero, f.err
        }
        
        return f.result, nil
        
    case <-ctx.Done():
        var zero T
        return zero, ctx.Err()
    }
}

// Usage
type ImageService struct {
    processor *ImageProcessor
}

func (s *ImageService) ProcessImageAsync(data []byte) *Future[Image] {
    future := NewFuture[Image]()
    
    go func() {
        result, err := s.processor.ProcessImage(data)
        if err != nil {
            future.CompleteExceptionally(err)
            return
        }
        future.Complete(result)
    }()
    
    return future
}
```

**Key Differences:**
- Java uses ExecutorService and CompletableFuture
- TypeScript leverages Promise integration
- Python implements with asyncio support
- Go uses channels and generics

**WHEN SHOULD YOU USE IT?**
- When operations are async
- When handling long computations
- When results come later
- When cancellation is needed
- When composing async tasks

**WHERE YOU'VE ALREADY SEEN IT**
- Promise in JavaScript
- CompletableFuture in Java
- Task in .NET
- Futures in Python
- Channel operations in Go
