# Aggregate Pattern

## Simple Explanation
Think of a car. You don't manage each part (wheels, engine, seats) independently - you manage them through the car itself. An Aggregate is like this - it's a cluster of related objects that we treat as one unit for data changes. The car is the "Aggregate Root" that ensures all parts change consistently.

## Real-World Scenario
Consider an e-commerce Order system:

Without Aggregates:
```typescript
class OrderService {
    async updateOrder(orderId: string, changes: any) {
        const order = await this.orderRepo.findById(orderId);
        const items = await this.itemRepo.findByOrderId(orderId);
        const payment = await this.paymentRepo.findByOrderId(orderId);
        
        // Dangerous: No consistency guarantee
        await this.itemRepo.updateItems(items, changes.items);
        await this.paymentRepo.updatePayment(payment, changes.payment);
        await this.orderRepo.update(order, changes.order);
    }
}
```

Problems:
- No consistency boundaries
- Multiple repositories
- Complex relationships
- No invariant protection
- Race conditions

With Aggregates:
```typescript
class Order {
    private items: OrderItem[];
    private payment: Payment;
    private status: OrderStatus;

    addItem(item: OrderItem) {
        if (this.status !== OrderStatus.Draft) {
            throw new Error('Cannot modify confirmed order');
        }
        if (this.items.length >= 10) {
            throw new Error('Maximum items reached');
        }
        this.items.push(item);
    }

    confirmOrder() {
        if (this.items.length === 0) {
            throw new Error('Order must have items');
        }
        if (!this.payment) {
            throw new Error('Payment required');
        }
        this.status = OrderStatus.Confirmed;
    }
}
```

Benefits:
- Single source of truth
- Protected invariants
- Consistent changes
- Clear boundaries
- Encapsulated rules

## Problem Statement
Without Aggregates, systems face:
- Inconsistent data changes
- Scattered business rules
- Complex relationships
- Race conditions
- No clear boundaries
- Difficult maintenance

## Solution
The Aggregate pattern provides:
1. Consistency Boundaries
   - One root entity
   - Single unit of change
   - Protected invariants

2. Encapsulation
   - Internal structure hidden
   - Business rules protected
   - Controlled access

3. Transaction Management
   - Atomic changes
   - Consistent state
   - Clear boundaries

## Implementation Examples

### TypeScript Implementation
```typescript
// Aggregate Root
class Order {
    private readonly id: string;
    private items: OrderItem[] = [];
    private payment?: Payment;
    private status: OrderStatus = OrderStatus.Draft;
    private version: number = 0;

    constructor(id: string) {
        this.id = id;
    }

    addItem(product: Product, quantity: number): void {
        if (this.status !== OrderStatus.Draft) {
            throw new OrderStateError('Cannot modify confirmed order');
        }

        const item = new OrderItem(product, quantity);
        if (this.getTotalItems() + quantity > 10) {
            throw new OrderLimitError('Maximum items exceeded');
        }

        this.items.push(item);
    }

    setPayment(payment: Payment): void {
        if (this.status !== OrderStatus.Draft) {
            throw new OrderStateError('Cannot modify payment after confirmation');
        }
        this.payment = payment;
    }

    confirmOrder(): void {
        if (this.items.length === 0) {
            throw new OrderValidationError('Order must have items');
        }
        if (!this.payment) {
            throw new OrderValidationError('Payment required');
        }
        this.status = OrderStatus.Confirmed;
        this.version++;
    }

    private getTotalItems(): number {
        return this.items.reduce((sum, item) => sum + item.quantity, 0);
    }
}

// Value Objects
class OrderItem {
    constructor(
        public readonly product: Product,
        public readonly quantity: number
    ) {
        if (quantity <= 0) {
            throw new ValidationError('Quantity must be positive');
        }
    }
}

// Repository
class OrderRepository {
    async save(order: Order): Promise<void> {
        const currentVersion = await this.getCurrentVersion(order.id);
        if (currentVersion !== order.version) {
            throw new ConcurrencyError('Order was modified');
        }
        await this.persist(order);
    }
}
```

### Java Implementation
```java
// Aggregate Root
@Entity
public class Order {
    @Id
    private final UUID id;
    
    @OneToMany(cascade = CascadeType.ALL)
    private List<OrderItem> items = new ArrayList<>();
    
    @OneToOne(cascade = CascadeType.ALL)
    private Payment payment;
    
    @Enumerated(EnumType.STRING)
    private OrderStatus status = OrderStatus.DRAFT;
    
    @Version
    private Long version;
    
    public void addItem(Product product, int quantity) {
        if (status != OrderStatus.DRAFT) {
            throw new OrderStateException("Cannot modify confirmed order");
        }
        
        OrderItem item = new OrderItem(product, quantity);
        if (getTotalItems() + quantity > 10) {
            throw new OrderLimitException("Maximum items exceeded");
        }
        
        items.add(item);
    }
    
    public void setPayment(Payment payment) {
        if (status != OrderStatus.DRAFT) {
            throw new OrderStateException(
                "Cannot modify payment after confirmation");
        }
        this.payment = payment;
    }
    
    public void confirmOrder() {
        if (items.isEmpty()) {
            throw new OrderValidationException("Order must have items");
        }
        if (payment == null) {
            throw new OrderValidationException("Payment required");
        }
        status = OrderStatus.CONFIRMED;
    }
    
    private int getTotalItems() {
        return items.stream()
            .mapToInt(OrderItem::getQuantity)
            .sum();
    }
}

// Value Objects
@Embeddable
public class OrderItem {
    @ManyToOne
    private final Product product;
    private final int quantity;
    
    public OrderItem(Product product, int quantity) {
        if (quantity <= 0) {
            throw new ValidationException("Quantity must be positive");
        }
        this.product = product;
        this.quantity = quantity;
    }
}

// Repository
@Repository
public class OrderRepository {
    @PersistenceContext
    private EntityManager em;
    
    @Transactional
    public void save(Order order) {
        em.persist(order);
    }
}
```

### Python Implementation
```python
from dataclasses import dataclass
from typing import List, Optional
from uuid import UUID, uuid4
from enum import Enum

class OrderStatus(Enum):
    DRAFT = "draft"
    CONFIRMED = "confirmed"

@dataclass
class OrderItem:
    product_id: UUID
    quantity: int

    def __post_init__(self):
        if self.quantity <= 0:
            raise ValueError("Quantity must be positive")

class Order:
    def __init__(self, id: UUID = None):
        self.id = id or uuid4()
        self.items: List[OrderItem] = []
        self.payment: Optional[Payment] = None
        self.status = OrderStatus.DRAFT
        self.version = 0

    def add_item(self, product_id: UUID, quantity: int) -> None:
        if self.status != OrderStatus.DRAFT:
            raise OrderStateError("Cannot modify confirmed order")

        item = OrderItem(product_id, quantity)
        if self.get_total_items() + quantity > 10:
            raise OrderLimitError("Maximum items exceeded")

        self.items.append(item)

    def set_payment(self, payment: Payment) -> None:
        if self.status != OrderStatus.DRAFT:
            raise OrderStateError(
                "Cannot modify payment after confirmation")
        self.payment = payment

    def confirm_order(self) -> None:
        if not self.items:
            raise OrderValidationError("Order must have items")
        if not self.payment:
            raise OrderValidationError("Payment required")
        
        self.status = OrderStatus.CONFIRMED
        self.version += 1

    def get_total_items(self) -> int:
        return sum(item.quantity for item in self.items)

class OrderRepository:
    def __init__(self, session):
        self.session = session

    async def save(self, order: Order) -> None:
        current = await self.get_version(order.id)
        if current != order.version:
            raise ConcurrencyError("Order was modified")
        
        await self.persist(order)
```

## Usage Guidelines

1. **When to Use**
   - Complex domain models
   - Important consistency rules
   - Multiple related objects
   - Transaction boundaries
   - Business invariants

2. **Best Practices**
   - Keep aggregates small
   - One transaction per aggregate
   - Reference other aggregates by ID
   - Protect invariants
   - Use value objects

3. **Implementation Considerations**
   - Concurrency control
   - Transaction boundaries
   - Loading strategy
   - Reference management
   - Event handling

## Real-World Examples

1. **E-commerce Systems**
   - Order aggregate
   - Shopping cart aggregate
   - Product catalog aggregate
   - Customer profile aggregate

2. **Banking Systems**
   - Account aggregate
   - Transaction aggregate
   - Customer aggregate
   - Loan application aggregate

3. **Inventory Systems**
   - Stock item aggregate
   - Purchase order aggregate
   - Supplier aggregate
   - Warehouse aggregate

## Key Implementation Differences

1. **TypeScript vs Java**
   - TypeScript: More functional approach
   - Java: JPA annotations
   - Both protect invariants

2. **Python vs Others**
   - Python: Dataclass usage
   - Simple value objects
   - Similar consistency rules

3. **Persistence Approaches**
   - TypeScript: Flexible storage
   - Java: JPA/Hibernate
   - Python: Custom persistence
