# Domain Event Pattern

## Simple Explanation
Think of a newspaper delivery system. When something important happens (like an earthquake), the newspaper creates a story (event) and all subscribers receive it. Similarly, Domain Events represent important things that happened in your domain, allowing other parts of the system to react accordingly.

## Real-World Scenario
Consider an e-commerce order system:

Without Domain Events:
```typescript
class OrderService {
    async confirmOrder(orderId: string) {
        const order = await this.orderRepo.findById(orderId);
        order.status = 'Confirmed';
        await this.orderRepo.save(order);
        
        // Direct coupling to all these services
        await this.inventoryService.reduceStock(order.items);
        await this.notificationService.sendConfirmation(order);
        await this.shippingService.scheduleDelivery(order);
        await this.analyticsService.trackOrder(order);
    }
}
```

Problems:
- Tight coupling
- Hard to extend
- Mixed responsibilities
- Complex testing
- Poor maintainability

With Domain Events:
```typescript
class Order {
    confirm() {
        if (!this.canBeConfirmed()) {
            throw new Error('Order cannot be confirmed');
        }
        
        this.status = 'Confirmed';
        this.addDomainEvent(new OrderConfirmedEvent(this));
    }
}

// Separate handlers
class InventoryHandler {
    @HandleEvent(OrderConfirmedEvent)
    async handle(event: OrderConfirmedEvent) {
        await this.inventoryService.reduceStock(event.order.items);
    }
}

class NotificationHandler {
    @HandleEvent(OrderConfirmedEvent)
    async handle(event: OrderConfirmedEvent) {
        await this.notificationService.sendConfirmation(event.order);
    }
}
```

Benefits:
- Loose coupling
- Single responsibility
- Easy to extend
- Better testability
- Clear dependencies

## Problem Statement
Without Domain Events, systems face:
- Tight coupling between components
- Mixed responsibilities
- Hard to extend functionality
- Complex testing scenarios
- Poor maintainability
- Unclear business flows

## Solution
The Domain Event pattern provides:
1. Clear Communication
   - Business events are explicit
   - Domain changes are tracked
   - Side effects are organized

2. Loose Coupling
   - Components are independent
   - Easy to add new handlers
   - Simple to test

3. Business Focus
   - Events represent domain changes
   - Clear business language
   - Explicit workflows

## Implementation Examples

### TypeScript Implementation
```typescript
// Base Event
interface DomainEvent {
    occurredOn: Date;
    eventType: string;
}

// Concrete Event
class OrderConfirmedEvent implements DomainEvent {
    public readonly occurredOn: Date;
    public readonly eventType: string = 'OrderConfirmed';

    constructor(public readonly order: Order) {
        this.occurredOn = new Date();
    }
}

// Event Emitting Entity
abstract class AggregateRoot {
    private domainEvents: DomainEvent[] = [];

    protected addDomainEvent(event: DomainEvent): void {
        this.domainEvents.push(event);
    }

    public clearDomainEvents(): DomainEvent[] {
        const events = [...this.domainEvents];
        this.domainEvents = [];
        return events;
    }
}

// Aggregate with Events
class Order extends AggregateRoot {
    confirm(): void {
        if (!this.canBeConfirmed()) {
            throw new Error('Order cannot be confirmed');
        }

        this.status = OrderStatus.Confirmed;
        this.addDomainEvent(new OrderConfirmedEvent(this));
    }
}

// Event Dispatcher
class DomainEventDispatcher {
    private handlers: Map<string, EventHandler[]> = new Map();

    register(eventType: string, handler: EventHandler): void {
        const handlers = this.handlers.get(eventType) || [];
        handlers.push(handler);
        this.handlers.set(eventType, handlers);
    }

    async dispatch(event: DomainEvent): Promise<void> {
        const handlers = this.handlers.get(event.eventType) || [];
        await Promise.all(
            handlers.map(handler => handler.handle(event))
        );
    }
}

// Unit of Work Integration
class UnitOfWork {
    private readonly dispatcher: DomainEventDispatcher;

    async commit(): Promise<void> {
        try {
            await this.commitTransaction();
            await this.dispatchDomainEvents();
        } catch (error) {
            await this.rollback();
            throw error;
        }
    }

    private async dispatchDomainEvents(): Promise<void> {
        const events = this.collectDomainEvents();
        for (const event of events) {
            await this.dispatcher.dispatch(event);
        }
    }
}
```

### Java Implementation
```java
// Base Event
public interface DomainEvent {
    LocalDateTime getOccurredOn();
    String getEventType();
}

// Concrete Event
@Value
public class OrderConfirmedEvent implements DomainEvent {
    Order order;
    LocalDateTime occurredOn = LocalDateTime.now();
    String eventType = "OrderConfirmed";
}

// Event Emitting Entity
public abstract class AggregateRoot {
    @Transient
    private List<DomainEvent> domainEvents = new ArrayList<>();

    protected void addDomainEvent(DomainEvent event) {
        domainEvents.add(event);
    }

    public List<DomainEvent> clearDomainEvents() {
        List<DomainEvent> events = new ArrayList<>(domainEvents);
        domainEvents.clear();
        return events;
    }
}

// Aggregate with Events
@Entity
public class Order extends AggregateRoot {
    public void confirm() {
        if (!canBeConfirmed()) {
            throw new IllegalStateException("Order cannot be confirmed");
        }

        this.status = OrderStatus.CONFIRMED;
        addDomainEvent(new OrderConfirmedEvent(this));
    }
}

// Event Dispatcher
@Service
public class DomainEventDispatcher {
    private final ApplicationEventPublisher publisher;

    public void dispatch(DomainEvent event) {
        publisher.publishEvent(event);
    }
}

// Event Handler
@Component
public class InventoryHandler {
    @EventListener
    public void handle(OrderConfirmedEvent event) {
        inventoryService.reduceStock(event.getOrder().getItems());
    }
}

// Transaction Management
@Service
@Transactional
public class OrderService {
    public void confirmOrder(UUID orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow();
        
        order.confirm();
        orderRepository.save(order);
        // Events dispatched after successful transaction
    }
}
```

### Python Implementation
```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from datetime import datetime
from typing import List, Dict, Type, Callable

# Base Event
class DomainEvent(ABC):
    @property
    @abstractmethod
    def occurred_on(self) -> datetime:
        pass

    @property
    @abstractmethod
    def event_type(self) -> str:
        pass

# Concrete Event
@dataclass
class OrderConfirmedEvent(DomainEvent):
    order: 'Order'
    occurred_on: datetime = field(default_factory=datetime.now)
    event_type: str = 'OrderConfirmed'

# Event Emitting Entity
class AggregateRoot:
    def __init__(self):
        self._domain_events: List[DomainEvent] = []

    def add_domain_event(self, event: DomainEvent) -> None:
        self._domain_events.append(event)

    def clear_domain_events(self) -> List[DomainEvent]:
        events = self._domain_events[:]
        self._domain_events.clear()
        return events

# Aggregate with Events
class Order(AggregateRoot):
    def confirm(self) -> None:
        if not self.can_be_confirmed():
            raise ValueError("Order cannot be confirmed")

        self.status = OrderStatus.CONFIRMED
        self.add_domain_event(OrderConfirmedEvent(self))

# Event Dispatcher
class DomainEventDispatcher:
    def __init__(self):
        self._handlers: Dict[str, List[Callable]] = {}

    def register(self, event_type: str, handler: Callable) -> None:
        if event_type not in self._handlers:
            self._handlers[event_type] = []
        self._handlers[event_type].append(handler)

    async def dispatch(self, event: DomainEvent) -> None:
        handlers = self._handlers.get(event.event_type, [])
        for handler in handlers:
            await handler(event)

# Unit of Work Integration
class UnitOfWork:
    def __init__(self, dispatcher: DomainEventDispatcher):
        self.dispatcher = dispatcher

    async def __aenter__(self):
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        if exc_type is None:
            await self.commit()
        else:
            await self.rollback()

    async def commit(self) -> None:
        try:
            await self._commit_transaction()
            await self._dispatch_domain_events()
        except Exception:
            await self.rollback()
            raise

    async def _dispatch_domain_events(self) -> None:
        events = self._collect_domain_events()
        for event in events:
            await self.dispatcher.dispatch(event)
```

## Usage Guidelines

1. **When to Use**
   - Complex domain logic
   - Multiple side effects
   - Audit requirements
   - Loose coupling needed
   - Event-driven architecture

2. **Best Practices**
   - Events are past tense
   - Include all relevant data
   - Make events immutable
   - Use meaningful names
   - Handle failures gracefully

3. **Implementation Considerations**
   - Event ordering
   - Transaction boundaries
   - Event persistence
   - Handler reliability
   - Error handling

## Real-World Examples

1. **E-commerce Systems**
   - OrderPlacedEvent
   - PaymentReceivedEvent
   - ShipmentScheduledEvent
   - OrderCancelledEvent

2. **Banking Systems**
   - TransactionCompletedEvent
   - AccountCreatedEvent
   - BalanceChangedEvent
   - LoanApprovedEvent

3. **Inventory Systems**
   - StockUpdatedEvent
   - ItemReservedEvent
   - LowStockEvent
   - RestockOrderedEvent

## Key Implementation Differences

1. **TypeScript vs Java**
   - TypeScript: Async event handling
   - Java: Spring events
   - Both support transaction boundaries

2. **Python vs Others**
   - Python: Async/await with context managers
   - Dataclass for events
   - Similar event dispatch pattern

3. **Transaction Handling**
   - TypeScript: Manual transaction-event coordination
   - Java: Spring transaction events
   - Python: Async context managers
