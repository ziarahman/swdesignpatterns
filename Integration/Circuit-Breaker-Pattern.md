# Circuit Breaker Pattern

## Simple Explanation
Think of a circuit breaker in your home's electrical system. When there's an overload, it trips to prevent damage. Similarly, the Circuit Breaker pattern prevents a software system from repeatedly trying to execute an operation that's likely to fail.

## Real-World Scenario
Imagine an e-commerce website that depends on a payment service. If the payment service is down:
- Without Circuit Breaker: The site keeps trying to process payments, causing delays and poor user experience
- With Circuit Breaker: The system quickly recognizes the failure and provides immediate feedback (e.g., "Payment service temporarily unavailable")

## Problem Statement
In distributed systems, calls to remote services can fail due to:
- Temporary network issues
- Service unavailability
- Timeout issues
These failures can cascade through the system, causing widespread issues and resource exhaustion.

## Solution
The Circuit Breaker pattern implements three states:
1. Closed (normal operation)
   - Requests pass through normally
   - Failures are counted
   - When failures exceed threshold, switch to Open
2. Open (failure state)
   - Requests fail fast without attempting operation
   - After timeout period, switch to Half-Open
3. Half-Open (testing recovery)
   - Allow limited requests through
   - If successful, switch to Closed
   - If fails, switch back to Open

## Implementation Examples

### TypeScript Implementation
```typescript
enum State {
    CLOSED,
    OPEN,
    HALF_OPEN
}

class CircuitBreaker {
    private state: State = State.CLOSED;
    private failureCount: number = 0;
    private lastFailureTime: number = 0;
    private readonly failureThreshold: number;
    private readonly resetTimeout: number;

    constructor(failureThreshold: number = 5, resetTimeout: number = 60000) {
        this.failureThreshold = failureThreshold;
        this.resetTimeout = resetTimeout;
    }

    async execute<T>(operation: () => Promise<T>): Promise<T> {
        if (!this.canExecute()) {
            throw new Error('Circuit breaker is OPEN');
        }

        try {
            const result = await operation();
            this.onSuccess();
            return result;
        } catch (error) {
            this.onFailure();
            throw error;
        }
    }

    private canExecute(): boolean {
        if (this.state === State.CLOSED) return true;
        if (this.state === State.OPEN) {
            if (Date.now() - this.lastFailureTime >= this.resetTimeout) {
                this.state = State.HALF_OPEN;
                return true;
            }
            return false;
        }
        return this.state === State.HALF_OPEN;
    }

    private onSuccess(): void {
        this.failureCount = 0;
        this.state = State.CLOSED;
    }

    private onFailure(): void {
        this.failureCount++;
        this.lastFailureTime = Date.now();

        if (this.failureCount >= this.failureThreshold) {
            this.state = State.OPEN;
        }
    }
}
```

### Java Implementation
```java
public class CircuitBreaker {
    private enum State { CLOSED, OPEN, HALF_OPEN }
    
    private State state;
    private int failureCount;
    private long lastFailureTime;
    private final int failureThreshold;
    private final long resetTimeout;
    
    public CircuitBreaker(int failureThreshold, long resetTimeout) {
        this.state = State.CLOSED;
        this.failureCount = 0;
        this.failureThreshold = failureThreshold;
        this.resetTimeout = resetTimeout;
    }
    
    public <T> T execute(Supplier<T> operation) throws Exception {
        if (!canExecute()) {
            throw new Exception("Circuit breaker is OPEN");
        }
        
        try {
            T result = operation.get();
            onSuccess();
            return result;
        } catch (Exception e) {
            onFailure();
            throw e;
        }
    }
    
    private boolean canExecute() {
        if (state == State.CLOSED) return true;
        if (state == State.OPEN) {
            if (System.currentTimeMillis() - lastFailureTime >= resetTimeout) {
                state = State.HALF_OPEN;
                return true;
            }
            return false;
        }
        return state == State.HALF_OPEN;
    }
    
    private void onSuccess() {
        failureCount = 0;
        state = State.CLOSED;
    }
    
    private void onFailure() {
        failureCount++;
        lastFailureTime = System.currentTimeMillis();
        
        if (failureCount >= failureThreshold) {
            state = State.OPEN;
        }
    }
}
```

### Python Implementation
```python
from enum import Enum
import time
from typing import TypeVar, Callable, Any
from functools import wraps

T = TypeVar('T')

class State(Enum):
    CLOSED = 'closed'
    OPEN = 'open'
    HALF_OPEN = 'half-open'

class CircuitBreaker:
    def __init__(self, failure_threshold: int = 5, reset_timeout: int = 60):
        self.state = State.CLOSED
        self.failure_count = 0
        self.last_failure_time = 0
        self.failure_threshold = failure_threshold
        self.reset_timeout = reset_timeout

    def __call__(self, func: Callable[..., T]) -> Callable[..., T]:
        @wraps(func)
        def wrapper(*args, **kwargs) -> T:
            if not self._can_execute():
                raise Exception("Circuit breaker is OPEN")

            try:
                result = func(*args, **kwargs)
                self._on_success()
                return result
            except Exception as e:
                self._on_failure()
                raise e

        return wrapper

    def _can_execute(self) -> bool:
        if self.state == State.CLOSED:
            return True
        if self.state == State.OPEN:
            if time.time() - self.last_failure_time >= self.reset_timeout:
                self.state = State.HALF_OPEN
                return True
            return False
        return self.state == State.HALF_OPEN

    def _on_success(self) -> None:
        self.failure_count = 0
        self.state = State.CLOSED

    def _on_failure(self) -> None:
        self.failure_count += 1
        self.last_failure_time = time.time()

        if self.failure_count >= self.failure_threshold:
            self.state = State.OPEN
```

## Usage Guidelines

1. **When to Use**
   - Remote service calls
   - Database operations
   - Third-party API integrations
   - Resource-intensive operations

2. **Configuration Parameters**
   - Failure Threshold: Number of failures before opening
   - Reset Timeout: Time to wait before attempting recovery
   - Failure Conditions: What constitutes a failure (timeouts, exceptions)

3. **Best Practices**
   - Set appropriate timeouts
   - Implement proper logging
   - Consider fallback mechanisms
   - Monitor circuit breaker states

## Real-World Examples

1. **Netflix Hystrix**
   - Used for latency and fault tolerance
   - Implements circuit breaker with fallback mechanisms
   - Provides metrics and monitoring

2. **Spring Cloud Circuit Breaker**
   - Abstraction across different circuit breaker implementations
   - Supports multiple backends (Resilience4J, Hystrix)
   - Integration with Spring Boot actuator

3. **Microsoft's Polly**
   - .NET resilience and transient-fault-handling library
   - Combines circuit breaker with retry policies
   - Supports timeout and bulkhead isolation

## Key Implementation Differences

1. **TypeScript vs Java**
   - TypeScript uses async/await for asynchronous operations
   - Java uses Supplier functional interface
   - Both maintain similar state management

2. **Python vs Others**
   - Python implements as a decorator
   - Uses function wrapping for easier integration
   - Similar state management but with Enum-based states

3. **Framework-Specific Features**
   - Spring: Integration with Spring Cloud
   - Node.js: Event-driven architecture support
   - .NET: Integration with dependency injection
