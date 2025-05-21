# Ambassador Pattern

## Simple Explanation
Think of a diplomatic ambassador who handles all communication between two countries, managing protocols, translations, and cultural differences. Similarly, the Ambassador pattern puts a proxy service in front of your application to handle common client-side integration tasks like monitoring, logging, routing, and security.

## Real-World Scenario
Consider a microservice connecting to a legacy database:

Without Ambassador:
```typescript
class ServiceClient {
    async connectToDatabase() {
        try {
            // Handle in application:
            // - Connection pooling
            // - Retry logic
            // - Circuit breaking
            // - Monitoring
            // - Authentication
            // - Protocol conversion
            const connection = await this.createConnection();
            return connection;
        } catch (error) {
            // Complex error handling
        }
    }
}
```

Problems:
- Complex client code
- Mixed responsibilities
- Duplicated across services
- Hard to update
- Difficult to maintain

With Ambassador:
```yaml
# Kubernetes deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: service-deployment
spec:
  containers:
  - name: main-service
    image: my-service:1.0
    # Simple connection to local ambassador
    env:
      - name: DB_HOST
        value: "localhost:5000"
  
  - name: db-ambassador
    image: db-ambassador:1.0
    # Handles all the complexity:
    # - Connection pooling
    # - Retries
    # - Circuit breaking
    # - Monitoring
    ports:
    - containerPort: 5000
```

Benefits:
- Simplified main service
- Reusable connectivity features
- Consistent patterns
- Easy to upgrade
- Better separation

## Problem Statement
Services need to handle:
- Complex connectivity
- Protocol translation
- Retry policies
- Circuit breaking
- Monitoring
- Security
But implementing these in each service is:
- Error-prone
- Duplicative
- Hard to maintain
- Inconsistent

## Solution
The Ambassador pattern provides:
1. Connection Management
   - Connection pooling
   - Retry logic
   - Circuit breaking
   - Load balancing

2. Cross-Cutting Concerns
   - Monitoring
   - Logging
   - Security
   - Protocol translation

3. Separation of Concerns
   - Main service focuses on business logic
   - Ambassador handles connectivity
   - Clear responsibilities

## Implementation Examples

### Kubernetes Implementation
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: service-with-ambassador
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: main-service
        image: my-service:1.0
        env:
        - name: REDIS_HOST
          value: "localhost:6379"
        
      - name: redis-ambassador
        image: redis-ambassador:1.0
        ports:
        - containerPort: 6379
        env:
        - name: REDIS_MASTER_HOST
          value: "redis-master.prod"
        - name: RETRY_COUNT
          value: "3"
        - name: CIRCUIT_BREAK_THRESHOLD
          value: "5"
```

### Docker Compose Implementation
```yaml
version: '3.8'
services:
  application:
    image: my-service:1.0
    environment:
      DB_HOST: localhost
      DB_PORT: 5432
    depends_on:
      - db-ambassador

  db-ambassador:
    image: db-ambassador:1.0
    environment:
      TARGET_HOST: db.production.svc
      TARGET_PORT: 5432
      MAX_CONNECTIONS: 10
      RETRY_INTERVAL: 5s
    ports:
      - "5432"
    volumes:
      - ./ambassador-config:/config

  metrics-ambassador:
    image: metrics-ambassador:1.0
    ports:
      - "9090"
```

### Implementation in Go
```go
package main

import (
    "context"
    "time"
    "net/http"
)

// Ambassador for HTTP services
type ServiceAmbassador struct {
    targetURL    string
    maxRetries   int
    timeout      time.Duration
    circuitBreaker *CircuitBreaker
}

func NewServiceAmbassador(targetURL string) *ServiceAmbassador {
    return &ServiceAmbassador{
        targetURL:    targetURL,
        maxRetries:   3,
        timeout:      time.Second * 10,
        circuitBreaker: NewCircuitBreaker(5, time.Minute),
    }
}

func (a *ServiceAmbassador) Call(ctx context.Context, request *http.Request) (*http.Response, error) {
    if !a.circuitBreaker.AllowRequest() {
        return nil, ErrCircuitOpen
    }

    for retry := 0; retry <= a.maxRetries; retry++ {
        client := &http.Client{Timeout: a.timeout}
        
        response, err := client.Do(request)
        if err == nil {
            a.circuitBreaker.RecordSuccess()
            return response, nil
        }

        a.circuitBreaker.RecordFailure()
        
        if retry == a.maxRetries {
            return nil, err
        }

        select {
        case <-ctx.Done():
            return nil, ctx.Err()
        case <-time.After(backoff(retry)):
            continue
        }
    }
    
    return nil, ErrMaxRetriesExceeded
}

// Circuit Breaker implementation
type CircuitBreaker struct {
    failureThreshold int
    resetTimeout     time.Duration
    failures         int
    lastFailure     time.Time
    state           State
}

func (cb *CircuitBreaker) AllowRequest() bool {
    if cb.state == Closed {
        return true
    }

    if time.Since(cb.lastFailure) > cb.resetTimeout {
        cb.state = HalfOpen
        return true
    }

    return false
}
```

## Usage Guidelines

1. **When to Use**
   - Complex service connections
   - Legacy system integration
   - Cross-cutting concerns
   - Protocol translation
   - Consistent patterns needed

2. **Best Practices**
   - Keep ambassador lightweight
   - Clear responsibility separation
   - Configure timeouts properly
   - Implement health checks
   - Monitor ambassador performance

3. **Implementation Considerations**
   - Connection management
   - Error handling
   - Resource allocation
   - Monitoring strategy
   - Security implications

## Real-World Examples

1. **Database Proxy**
   - Connection pooling
   - Query routing
   - Failover handling
   - Read/write splitting
   - Query monitoring

2. **Redis Ambassador**
   - Connection management
   - Retry handling
   - Cluster routing
   - Monitoring
   - Error handling

3. **Service Mesh Proxy**
   - Traffic management
   - Security
   - Observability
   - Policy enforcement
   - Protocol translation

## Key Implementation Differences

1. **Kubernetes vs Docker Compose**
   - Kubernetes: Pod-based colocation
   - Docker Compose: Service-based
   - Different networking models

2. **Protocol Specific**
   - HTTP: REST/GraphQL handling
   - Database: Connection pooling
   - Cache: Cluster management
   - Different optimization needs

3. **Deployment Options**
   - Sidecar deployment
   - Dedicated service
   - Different scaling needs
