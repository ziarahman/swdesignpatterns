# Anti-Corruption Layer Pattern

## Simple Explanation
Think of a universal translator in science fiction that allows different alien species to communicate without losing their own language and culture. The Anti-Corruption Layer (ACL) pattern works similarly - it's a layer that translates between different system models while protecting your domain model from external influences and legacy concepts.

## Real-World Scenario
Consider integrating a modern e-commerce system with a legacy inventory system:

Without ACL:
```typescript
class OrderService {
    async createOrder(order: ModernOrder) {
        // Modern domain corrupted by legacy concepts
        const legacyItems = order.items.map(item => ({
            PROD_ID: item.id,
            QTY: item.quantity,
            WHSE_LOC: 'DEFAULT',
            UPD_DT: new Date().toISOString()
        }));

        // Direct dependency on legacy system
        await this.legacyInventorySystem.updateInventory({
            TXN_TYPE: 'ORD',
            TXN_DATE: new Date(),
            ITEMS: legacyItems
        });
    }
}
```

Problems:
- Domain model pollution
- Legacy dependencies
- Complex translations
- Hard to maintain
- Difficult to test

With ACL:
```typescript
// Modern domain model stays clean
class Order {
    constructor(
        public id: string,
        public items: OrderItem[]
    ) {}

    async process() {
        await this.orderProcessor.process(this);
    }
}

// ACL handles all legacy integration
class LegacyInventoryACL {
    async updateInventory(order: Order) {
        const legacyFormat = this.translateToLegacy(order);
        await this.legacyClient.processTransaction(legacyFormat);
    }

    private translateToLegacy(order: Order): LegacyTransaction {
        // Translation happens here, isolated from domain
        return {
            TXN_TYPE: 'ORD',
            TXN_DATE: new Date(),
            ITEMS: order.items.map(this.translateItem)
        };
    }
}
```

Benefits:
- Clean domain model
- Isolated legacy code
- Clear translations
- Easy to maintain
- Simple to test

## Problem Statement
When integrating with legacy or external systems:
- Domain models get corrupted
- Business logic gets mixed
- Complex translations scattered
- Hard to maintain
- Difficult to evolve
- Testing becomes complex

## Solution
The Anti-Corruption Layer pattern provides:
1. Model Protection
   - Domain model isolation
   - Clean boundaries
   - Clear translations

2. Legacy Integration
   - Encapsulated translations
   - Protocol adaptation
   - Format conversion

3. Evolution Support
   - Easy to change
   - Simple to test
   - Clear responsibilities

## Implementation Examples

### TypeScript Implementation
```typescript
// Modern Domain Model
interface OrderItem {
    productId: string;
    quantity: number;
    price: number;
}

class Order {
    constructor(
        public readonly id: string,
        public readonly items: OrderItem[]
    ) {}

    get totalAmount(): number {
        return this.items.reduce(
            (sum, item) => sum + item.price * item.quantity, 
            0
        );
    }
}

// Legacy System Interface
interface LegacyInventorySystem {
    updateInventory(data: {
        TXN_TYPE: string;
        TXN_DATE: Date;
        ITEMS: Array<{
            PROD_ID: string;
            QTY: number;
            WHSE_LOC: string;
            UPD_DT: string;
        }>;
    }): Promise<void>;
}

// Anti-Corruption Layer
class InventoryACL {
    constructor(
        private legacySystem: LegacyInventorySystem
    ) {}

    async processOrder(order: Order): Promise<void> {
        const legacyFormat = this.translateToLegacy(order);
        await this.legacySystem.updateInventory(legacyFormat);
    }

    private translateToLegacy(order: Order) {
        return {
            TXN_TYPE: 'ORD',
            TXN_DATE: new Date(),
            ITEMS: order.items.map(item => ({
                PROD_ID: item.productId,
                QTY: item.quantity,
                WHSE_LOC: 'DEFAULT',
                UPD_DT: new Date().toISOString()
            }))
        };
    }
}

// Modern Service
class OrderService {
    constructor(private inventoryACL: InventoryACL) {}

    async processOrder(order: Order): Promise<void> {
        // Domain logic stays clean
        if (order.totalAmount > 1000) {
            throw new Error('Order amount too high');
        }

        // Legacy integration handled through ACL
        await this.inventoryACL.processOrder(order);
    }
}
```

### Java Implementation
```java
// Modern Domain Model
@Value
public class Order {
    String id;
    List<OrderItem> items;

    public BigDecimal getTotalAmount() {
        return items.stream()
            .map(item -> item.getPrice()
                .multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}

// Legacy System Interface
public interface LegacyInventorySystem {
    void updateInventory(LegacyTransaction transaction) 
        throws LegacySystemException;
}

// Anti-Corruption Layer
@Service
public class InventoryACL {
    private final LegacyInventorySystem legacySystem;
    private final TransactionMapper mapper;

    public void processOrder(Order order) {
        LegacyTransaction legacyTxn = mapper.toTransaction(order);
        try {
            legacySystem.updateInventory(legacyTxn);
        } catch (LegacySystemException e) {
            throw new InventoryException(
                "Failed to update inventory", e);
        }
    }

    @Component
    class TransactionMapper {
        LegacyTransaction toTransaction(Order order) {
            return LegacyTransaction.builder()
                .txnType("ORD")
                .txnDate(LocalDateTime.now())
                .items(mapItems(order.getItems()))
                .build();
        }

        private List<LegacyItem> mapItems(List<OrderItem> items) {
            return items.stream()
                .map(this::mapItem)
                .collect(Collectors.toList());
        }
    }
}
```

### Python Implementation
```python
from dataclasses import dataclass
from datetime import datetime
from typing import List, Dict, Any
from decimal import Decimal

# Modern Domain Model
@dataclass
class OrderItem:
    product_id: str
    quantity: int
    price: Decimal

@dataclass
class Order:
    id: str
    items: List[OrderItem]

    @property
    def total_amount(self) -> Decimal:
        return sum(
            item.price * item.quantity 
            for item in self.items
        )

# Legacy System Interface
class LegacyInventorySystem:
    async def update_inventory(
        self, 
        transaction: Dict[str, Any]
    ) -> None:
        # Legacy system call
        pass

# Anti-Corruption Layer
class InventoryACL:
    def __init__(self, legacy_system: LegacyInventorySystem):
        self.legacy_system = legacy_system

    async def process_order(self, order: Order) -> None:
        legacy_format = self._translate_to_legacy(order)
        try:
            await self.legacy_system.update_inventory(
                legacy_format
            )
        except Exception as e:
            raise InventoryException(
                f"Failed to update inventory: {str(e)}"
            )

    def _translate_to_legacy(self, order: Order) -> Dict[str, Any]:
        return {
            "TXN_TYPE": "ORD",
            "TXN_DATE": datetime.now(),
            "ITEMS": [
                {
                    "PROD_ID": item.product_id,
                    "QTY": item.quantity,
                    "WHSE_LOC": "DEFAULT",
                    "UPD_DT": datetime.now().isoformat()
                }
                for item in order.items
            ]
        }
```

## Usage Guidelines

1. **When to Use**
   - Legacy system integration
   - External service adaptation
   - Model protection needed
   - Complex translations
   - Domain evolution

2. **Best Practices**
   - Keep translations isolated
   - Use clear interfaces
   - Handle errors gracefully
   - Unit test translations
   - Document legacy quirks

3. **Implementation Considerations**
   - Translation complexity
   - Error handling strategy
   - Performance impact
   - Maintenance approach
   - Testing strategy

## Real-World Examples

1. **Banking Systems**
   - Modern API to legacy mainframe
   - Payment system integration
   - Account system modernization
   - Regulatory reporting

2. **E-commerce**
   - Modern frontend to legacy backend
   - Inventory system integration
   - Payment gateway adaptation
   - Shipping system integration

3. **Healthcare**
   - Modern EMR to legacy systems
   - Insurance system integration
   - Lab system adaptation
   - Regulatory compliance

## Key Implementation Differences

1. **TypeScript vs Java**
   - TypeScript: Functional approach
   - Java: Object mapping
   - Both protect domain model

2. **Python vs Others**
   - Python: Dataclass usage
   - Async integration
   - Similar translation patterns

3. **Error Handling**
   - TypeScript: Promise rejection
   - Java: Exception translation
   - Python: Exception handling
