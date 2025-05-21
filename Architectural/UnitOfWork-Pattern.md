# Design Patterns: Architectural - Unit of Work

## Unit of Work Pattern - Simple Explanation

**SITUATION: YOU'RE MANAGING TRANSACTIONS**  
You have multiple repositories modifying data. 

Initially, you have: 

#### TypeScript
```typescript
class OrderService {
    async createOrder(order: Order) {
        await this.orderRepo.save(order);
        await this.inventoryRepo.updateStock(order.items);
    }
}
``` 
All good.

**THEN THE COMPLEXITY GROWS**  
Now you need:  
- Transaction management
- Multiple repositories
- Coordinated updates
- Failure handling
- Consistent state

You're stuck with: 
```typescript
class OrderService {
    async createOrder(order: Order) {
        const transaction = await db.beginTransaction();
        try {
            await this.orderRepo.save(order);
            await this.inventoryRepo.updateStock(order.items);
            await this.paymentRepo.processPayment(order.payment);
            await transaction.commit();
        } catch (error) {
            await transaction.rollback();
            throw error;
        }
    }
}
``` 
Transaction logic everywhere!

**WHAT'S THE PROBLEM?**  
- Transaction management scattered
- Hard to ensure consistency
- Complex error handling
- Duplicate transaction code
- No unified approach

**HOW UNIT OF WORK SAVES YOU**  
Create a coordinator for all data changes.

You say: "Track all changes and commit them as one unit." 

And the Unit of Work:  
- Tracks changes
- Manages transactions
- Ensures consistency
- Handles rollbacks
- Coordinates repositories

Now you just: 
```typescript
async createOrder(order: Order) {
    using(const uow = new UnitOfWork()) {
        await uow.orders.add(order);
        await uow.inventory.updateStock(order.items);
        await uow.commit();
    }
}
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### TypeScript  
```typescript  
interface IUnitOfWork {
    orders: IOrderRepository;
    inventory: IInventoryRepository;
    commit(): Promise<void>;
    rollback(): Promise<void>;
}

class UnitOfWork implements IUnitOfWork {
    private readonly transaction: Transaction;
    
    constructor(
        public readonly orders: IOrderRepository,
        public readonly inventory: IInventoryRepository
    ) {
        this.transaction = new Transaction();
    }
    
    async commit(): Promise<void> {
        try {
            await this.transaction.begin();
            await this.transaction.commit();
        } catch (error) {
            await this.rollback();
            throw error;
        }
    }
    
    async rollback(): Promise<void> {
        await this.transaction.rollback();
    }
}

class OrderService {
    constructor(private uow: IUnitOfWork) {}
    
    async createOrder(order: Order): Promise<void> {
        try {
            await this.uow.orders.add(order);
            await this.uow.inventory.updateStock(order.items);
            await this.uow.commit();
        } catch (error) {
            await this.uow.rollback();
            throw error;
        }
    }
}
```

#### C#  
```csharp  
public interface IUnitOfWork : IDisposable
{
    IOrderRepository Orders { get; }
    IInventoryRepository Inventory { get; }
    Task CommitAsync();
}

public class UnitOfWork : IUnitOfWork
{
    private readonly DbContext _context;
    private bool _disposed = false;
    
    public IOrderRepository Orders { get; }
    public IInventoryRepository Inventory { get; }
    
    public UnitOfWork(DbContext context)
    {
        _context = context;
        Orders = new OrderRepository(_context);
        Inventory = new InventoryRepository(_context);
    }
    
    public async Task CommitAsync()
    {
        using (var transaction = await _context.Database.BeginTransactionAsync())
        {
            try
            {
                await _context.SaveChangesAsync();
                await transaction.CommitAsync();
            }
            catch
            {
                await transaction.RollbackAsync();
                throw;
            }
        }
    }
    
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed && disposing)
        {
            _context.Dispose();
        }
        _disposed = true;
    }
    
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
}

public class OrderService
{
    private readonly IUnitOfWork _unitOfWork;
    
    public OrderService(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }
    
    public async Task CreateOrderAsync(Order order)
    {
        await _unitOfWork.Orders.AddAsync(order);
        await _unitOfWork.Inventory.UpdateStockAsync(order.Items);
        await _unitOfWork.CommitAsync();
    }
}
```

#### Python  
```python  
from abc import ABC, abstractmethod
from contextlib import AbstractContextManager
from typing import Optional
from sqlalchemy.orm import Session

class UnitOfWork(AbstractContextManager):
    def __init__(self, session_factory):
        self.session_factory = session_factory
        self.session: Optional[Session] = None
    
    def __enter__(self):
        self.session = self.session_factory()
        self.orders = OrderRepository(self.session)
        self.inventory = InventoryRepository(self.session)
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is not None:
            self.rollback()
        self.session.close()
    
    def commit(self):
        self.session.commit()
    
    def rollback(self):
        self.session.rollback()

class OrderService:
    def __init__(self, uow: UnitOfWork):
        self.uow = uow
    
    def create_order(self, order: Order):
        with self.uow as uow:
            try:
                uow.orders.add(order)
                uow.inventory.update_stock(order.items)
                uow.commit()
            except Exception as e:
                uow.rollback()
                raise e
```

#### Go  
```go  
package main

import (
    "context"
    "database/sql"
)

type UnitOfWork struct {
    db *sql.DB
    tx *sql.Tx
    
    Orders    OrderRepository
    Inventory InventoryRepository
}

func NewUnitOfWork(db *sql.DB) (*UnitOfWork, error) {
    tx, err := db.Begin()
    if err != nil {
        return nil, err
    }
    
    return &UnitOfWork{
        db: db,
        tx: tx,
        Orders:    NewOrderRepository(tx),
        Inventory: NewInventoryRepository(tx),
    }, nil
}

func (uow *UnitOfWork) Commit() error {
    return uow.tx.Commit()
}

func (uow *UnitOfWork) Rollback() error {
    return uow.tx.Rollback()
}

type OrderService struct {
    db *sql.DB
}

func (s *OrderService) CreateOrder(ctx context.Context, order *Order) error {
    uow, err := NewUnitOfWork(s.db)
    if err != nil {
        return err
    }
    
    defer func() {
        if r := recover(); r != nil {
            uow.Rollback()
            panic(r)
        }
    }()
    
    if err := uow.Orders.Create(ctx, order); err != nil {
        uow.Rollback()
        return err
    }
    
    if err := uow.Inventory.UpdateStock(ctx, order.Items); err != nil {
        uow.Rollback()
        return err
    }
    
    return uow.Commit()
}
```

**Key Differences:**  
- TypeScript uses async/await with transactions
- C# leverages DbContext and IDisposable
- Python uses context managers
- Go implements with sql.Tx and defer

**WHEN SHOULD YOU USE IT?**  
- When managing multiple repositories
- When ensuring data consistency
- When handling transactions
- When coordinating multiple operations

**WHERE YOU'VE ALREADY SEEN IT**  
- ORM frameworks
- Database abstractions
- Enterprise applications
- Distributed systems
