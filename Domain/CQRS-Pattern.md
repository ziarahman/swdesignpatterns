# CQRS Pattern (Command Query Responsibility Segregation)

## Simple Explanation
Think of a restaurant with separate order-taking staff and food runners. The waiters who take orders (Commands) are different from those who check on food status (Queries). This separation allows each team to optimize for their specific task. CQRS works the same way - separating the systems that handle data changes from those that handle data reads.

## Real-World Scenario
Consider an e-commerce system:

Without CQRS:
```typescript
class OrderService {
    async createOrder(order: Order) {
        return await this.repository.save(order);
    }
    
    async getOrder(id: string) {
        return await this.repository.findById(id);
    }
    
    async getOrdersWithDetails() {
        return await this.repository.findAllWithJoins();
    }
}
```

The problems:
- Complex queries slow down writes
- Write operations block reads
- One model for all operations
- Poor scalability
- Complex optimization

With CQRS:
```typescript
// Command Side (Writes)
class OrderCommandService {
    async createOrder(order: Order) {
        const event = new OrderCreatedEvent(order);
        await this.eventStore.append(event);
        await this.commandRepository.save(order);
    }
}

// Query Side (Reads)
class OrderQueryService {
    async getOrderDetails(id: string) {
        return await this.readModelRepository
            .findByIdWithAllDetails(id);
    }
}
```

Now you have:
- Optimized read and write models
- Independent scaling
- Simpler queries
- Better performance
- Easier caching

## Problem Statement
Traditional CRUD applications face:
- Complex queries impact write performance
- Single model for reads and writes
- Difficult optimization
- Limited scalability
- Complex caching
- Conflicting requirements

## Solution
CQRS provides:
1. Separation of Concerns
   - Command model for writes
   - Query model for reads
   - Different storage optimizations

2. Scalability
   - Scale reads independently
   - Scale writes independently
   - Optimize for each use case

3. Performance
   - Specialized data models
   - Efficient queries
   - Better caching
   - Reduced contention

## Implementation Examples

### TypeScript Implementation
```typescript
// Command Side
interface Command {
    type: string;
    data: any;
}

class CreateOrderCommand implements Command {
    type = 'CreateOrder';
    constructor(public data: Order) {}
}

class OrderCommandHandler {
    constructor(
        private eventStore: EventStore,
        private writeRepository: WriteRepository
    ) {}

    async handle(command: CreateOrderCommand): Promise<void> {
        // Validate command
        const order = command.data;
        
        // Create and store event
        const event = new OrderCreatedEvent(order);
        await this.eventStore.append(event);
        
        // Update write model
        await this.writeRepository.save(order);
    }
}

// Query Side
interface OrderDetails {
    id: string;
    items: OrderItem[];
    customer: CustomerInfo;
    status: OrderStatus;
}

class OrderQueryService {
    constructor(private readRepository: ReadRepository) {}

    async getOrderDetails(id: string): Promise<OrderDetails> {
        return this.readRepository.findOrderDetails(id);
    }

    async getOrdersByCustomer(customerId: string): Promise<OrderSummary[]> {
        return this.readRepository.findByCustomer(customerId);
    }
}

// Event Handler to Keep Read Model Updated
class OrderEventHandler {
    constructor(private readRepository: ReadRepository) {}

    async handle(event: OrderCreatedEvent): Promise<void> {
        await this.readRepository.updateReadModel(event);
    }
}
```

### Java Implementation
```java
// Command Side
public interface Command {
    String getType();
    Object getData();
}

@Value
public class CreateOrderCommand implements Command {
    String type = "CreateOrder";
    Order data;
}

@Service
public class OrderCommandHandler {
    private final EventStore eventStore;
    private final WriteRepository writeRepository;
    
    public void handle(CreateOrderCommand command) {
        // Validate command
        Order order = command.getData();
        
        // Create and store event
        OrderCreatedEvent event = new OrderCreatedEvent(order);
        eventStore.append(event);
        
        // Update write model
        writeRepository.save(order);
    }
}

// Query Side
@Value
public class OrderDetails {
    String id;
    List<OrderItem> items;
    CustomerInfo customer;
    OrderStatus status;
}

@Service
public class OrderQueryService {
    private final ReadRepository readRepository;
    
    public OrderDetails getOrderDetails(String id) {
        return readRepository.findOrderDetails(id);
    }
    
    public List<OrderSummary> getOrdersByCustomer(String customerId) {
        return readRepository.findByCustomer(customerId);
    }
}

// Event Handler
@Service
public class OrderEventHandler {
    private final ReadRepository readRepository;
    
    @EventListener
    public void handle(OrderCreatedEvent event) {
        readRepository.updateReadModel(event);
    }
}
```

### Python Implementation
```python
from dataclasses import dataclass
from typing import List, Optional
from abc import ABC, abstractmethod

# Command Side
@dataclass
class Command:
    type: str
    data: dict

class CreateOrderCommand(Command):
    def __init__(self, order_data: dict):
        super().__init__('CreateOrder', order_data)

class OrderCommandHandler:
    def __init__(self, event_store, write_repository):
        self.event_store = event_store
        self.write_repository = write_repository

    async def handle(self, command: CreateOrderCommand) -> None:
        # Validate command
        order = command.data
        
        # Create and store event
        event = OrderCreatedEvent(order)
        await self.event_store.append(event)
        
        # Update write model
        await self.write_repository.save(order)

# Query Side
@dataclass
class OrderDetails:
    id: str
    items: List['OrderItem']
    customer: 'CustomerInfo'
    status: str

class OrderQueryService:
    def __init__(self, read_repository):
        self.read_repository = read_repository

    async def get_order_details(self, id: str) -> OrderDetails:
        return await self.read_repository.find_order_details(id)

    async def get_orders_by_customer(
        self, customer_id: str
    ) -> List['OrderSummary']:
        return await self.read_repository.find_by_customer(customer_id)

# Event Handler
class OrderEventHandler:
    def __init__(self, read_repository):
        self.read_repository = read_repository

    async def handle(self, event: 'OrderCreatedEvent') -> None:
        await self.read_repository.update_read_model(event)
```

## Usage Guidelines

1. **When to Use**
   - Complex read operations
   - Different read/write requirements
   - High scalability needs
   - Performance-critical systems
   - Event-sourced systems

2. **Best Practices**
   - Start simple, evolve to CQRS
   - Consider eventual consistency
   - Plan for data synchronization
   - Monitor read model latency
   - Handle edge cases

3. **Implementation Considerations**
   - Eventual consistency strategy
   - Read model update approach
   - Command validation
   - Error handling
   - Data synchronization

## Real-World Examples

1. **E-commerce Platforms**
   - Write model: Orders, inventory
   - Read model: Product catalog, order history
   - Different scaling needs
   - Complex reporting queries

2. **Financial Systems**
   - Write model: Transactions
   - Read model: Account balances, reports
   - High consistency needs
   - Complex aggregations

3. **Social Media Platforms**
   - Write model: Posts, interactions
   - Read model: Feeds, timelines
   - High read scalability
   - Complex content queries

## Key Implementation Differences

1. **TypeScript vs Java**
   - TypeScript: Async/await pattern
   - Java: Spring event system
   - Both support event sourcing

2. **Python vs Others**
   - Python: Asyncio for async
   - Dataclass for models
   - Similar separation pattern

3. **Architecture Choices**
   - Event-driven vs Direct updates
   - Sync vs Async propagation
   - Storage technology choices
