# Event Sourcing Pattern

## Simple Explanation
Think of Event Sourcing like your bank account statement. Instead of just knowing your current balance, you have a complete record of every deposit and withdrawal that led to it. Similarly, Event Sourcing stores the complete history of how your data reached its current state through a series of events.

## Real-World Scenario
Consider an e-commerce order:

Without Event Sourcing:
```typescript
class Order {
    status: string;
    updateStatus(newStatus: string) {
        this.status = newStatus; // Previous state is lost
    }
}
```

With Event Sourcing:
```typescript
class Order {
    events: OrderEvent[] = [];
    
    updateStatus(newStatus: string) {
        this.events.push(new StatusChangedEvent(this.status, newStatus));
    }
    
    get currentStatus() {
        return this.events
            .filter(e => e instanceof StatusChangedEvent)
            .reduce((_, e) => e.newStatus, 'created');
    }
}
```

Now you can:
- See complete order history
- Understand state changes
- Audit everything
- Replay events
- Debug issues

## Problem Statement
Traditional CRUD systems face:
- Lost historical context
- No audit trail
- Hard to debug
- Difficult to test
- Complex temporal queries
- Compliance challenges

## Solution
Event Sourcing provides:
1. Complete History
   - All changes recorded as events
   - Nothing is ever deleted
   - Full audit capability

2. State Recovery
   - Rebuild state from events
   - Point-in-time reconstruction
   - Temporal queries

3. Event-Driven Architecture
   - Natural event publishing
   - Easy integration
   - Scalable design

## Implementation Examples

### TypeScript Implementation
```typescript
interface Event {
    id: string;
    timestamp: Date;
    type: string;
    data: any;
}

class EventStore {
    private events: Event[] = [];
    private subscribers: ((event: Event) => void)[] = [];

    async append(event: Event): Promise<void> {
        this.events.push(event);
        this.subscribers.forEach(sub => sub(event));
    }

    async getEvents(aggregateId: string): Promise<Event[]> {
        return this.events.filter(e => e.data.aggregateId === aggregateId);
    }

    subscribe(callback: (event: Event) => void): void {
        this.subscribers.push(callback);
    }
}

class Order {
    private state: {
        id: string;
        status: string;
        items: any[];
    };

    constructor(private eventStore: EventStore, private id: string) {
        this.state = { id, status: 'created', items: [] };
    }

    async load(): Promise<void> {
        const events = await this.eventStore.getEvents(this.id);
        events.forEach(event => this.apply(event));
    }

    async addItem(item: any): Promise<void> {
        const event: Event = {
            id: crypto.randomUUID(),
            timestamp: new Date(),
            type: 'ItemAdded',
            data: { aggregateId: this.id, item }
        };
        await this.eventStore.append(event);
        this.apply(event);
    }

    async updateStatus(status: string): Promise<void> {
        const event: Event = {
            id: crypto.randomUUID(),
            timestamp: new Date(),
            type: 'StatusUpdated',
            data: { aggregateId: this.id, status }
        };
        await this.eventStore.append(event);
        this.apply(event);
    }

    private apply(event: Event): void {
        switch (event.type) {
            case 'ItemAdded':
                this.state.items.push(event.data.item);
                break;
            case 'StatusUpdated':
                this.state.status = event.data.status;
                break;
        }
    }
}
```

### Java Implementation
```java
public interface Event {
    UUID getId();
    LocalDateTime getTimestamp();
    String getType();
    Map<String, Object> getData();
}

@RequiredArgsConstructor
public class EventStore {
    private final List<Event> events = new ArrayList<>();
    private final List<Consumer<Event>> subscribers = new ArrayList<>();

    public void append(Event event) {
        events.add(event);
        subscribers.forEach(sub -> sub.accept(event));
    }

    public List<Event> getEvents(UUID aggregateId) {
        return events.stream()
            .filter(e -> e.getData().get("aggregateId").equals(aggregateId))
            .collect(Collectors.toList());
    }

    public void subscribe(Consumer<Event> callback) {
        subscribers.add(callback);
    }
}

public class Order {
    private final EventStore eventStore;
    private final UUID id;
    private OrderState state;

    public Order(EventStore eventStore, UUID id) {
        this.eventStore = eventStore;
        this.id = id;
        this.state = new OrderState(id);
    }

    public void load() {
        eventStore.getEvents(id)
            .forEach(this::apply);
    }

    public void addItem(OrderItem item) {
        Event event = OrderEvent.builder()
            .id(UUID.randomUUID())
            .timestamp(LocalDateTime.now())
            .type("ItemAdded")
            .data(Map.of(
                "aggregateId", id,
                "item", item
            ))
            .build();
            
        eventStore.append(event);
        apply(event);
    }

    public void updateStatus(String status) {
        Event event = OrderEvent.builder()
            .id(UUID.randomUUID())
            .timestamp(LocalDateTime.now())
            .type("StatusUpdated")
            .data(Map.of(
                "aggregateId", id,
                "status", status
            ))
            .build();
            
        eventStore.append(event);
        apply(event);
    }

    private void apply(Event event) {
        switch (event.getType()) {
            case "ItemAdded":
                state.getItems().add((OrderItem)event.getData().get("item"));
                break;
            case "StatusUpdated":
                state.setStatus((String)event.getData().get("status"));
                break;
        }
    }
}
```

### Python Implementation
```python
from dataclasses import dataclass
from datetime import datetime
from typing import Any, Dict, List, Callable
from uuid import UUID, uuid4

@dataclass
class Event:
    id: UUID
    timestamp: datetime
    type: str
    data: Dict[str, Any]

class EventStore:
    def __init__(self):
        self.events: List[Event] = []
        self.subscribers: List[Callable[[Event], None]] = []

    async def append(self, event: Event) -> None:
        self.events.append(event)
        for sub in self.subscribers:
            await sub(event)

    async def get_events(self, aggregate_id: UUID) -> List[Event]:
        return [
            e for e in self.events 
            if e.data.get('aggregate_id') == aggregate_id
        ]

    def subscribe(self, callback: Callable[[Event], None]) -> None:
        self.subscribers.append(callback)

class Order:
    def __init__(self, event_store: EventStore, id: UUID):
        self.event_store = event_store
        self.id = id
        self.state = {'id': id, 'status': 'created', 'items': []}

    async def load(self) -> None:
        events = await self.event_store.get_events(self.id)
        for event in events:
            self._apply(event)

    async def add_item(self, item: Dict[str, Any]) -> None:
        event = Event(
            id=uuid4(),
            timestamp=datetime.now(),
            type='ItemAdded',
            data={'aggregate_id': self.id, 'item': item}
        )
        await self.event_store.append(event)
        self._apply(event)

    async def update_status(self, status: str) -> None:
        event = Event(
            id=uuid4(),
            timestamp=datetime.now(),
            type='StatusUpdated',
            data={'aggregate_id': self.id, 'status': status}
        )
        await self.event_store.append(event)
        self._apply(event)

    def _apply(self, event: Event) -> None:
        if event.type == 'ItemAdded':
            self.state['items'].append(event.data['item'])
        elif event.type == 'StatusUpdated':
            self.state['status'] = event.data['status']
```

## Usage Guidelines

1. **When to Use**
   - Audit requirements
   - Complex domain logic
   - Temporal queries needed
   - High data integrity needs
   - Event-driven systems

2. **Best Practices**
   - Events are immutable
   - Store raw events
   - Use snapshots for performance
   - Consider event versioning
   - Plan for event schema evolution

3. **Implementation Considerations**
   - Event storage strategy
   - Snapshot frequency
   - Concurrency handling
   - Event versioning
   - Performance optimization

## Real-World Examples

1. **Version Control Systems**
   - Git stores complete history
   - Commits are events
   - State is reconstructible
   - Branch management

2. **Accounting Systems**
   - Journal entries as events
   - Balance sheet from events
   - Audit trail built-in
   - Regulatory compliance

3. **E-commerce Platforms**
   - Order status tracking
   - Inventory management
   - Customer activity history
   - Shipping status updates

## Key Implementation Differences

1. **TypeScript vs Java**
   - TypeScript uses async/await
   - Java uses CompletableFuture
   - Both maintain event stores

2. **Python vs Others**
   - Python uses asyncio
   - Dataclass for events
   - Similar event store pattern

3. **State Management**
   - TypeScript: Functional approach
   - Java: OOP with builders
   - Python: Dictionary-based state
