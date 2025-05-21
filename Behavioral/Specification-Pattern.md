# Design Patterns: Behavioral - Specification

## Specification Pattern - Simple Explanation

**SITUATION: YOU'RE FILTERING DATA**  
You have a collection of products you need to filter. 

Initially, you have: 

#### Java
```java
products.stream()
    .filter(p -> p.getPrice() > 100)
    .collect(Collectors.toList());
``` 
All good.

**THEN THE FILTERS GROW**  
Now you have complex business rules:  
- Premium products
- In-stock items
- Seasonal offers
- Price ranges
- Multiple combinations

You're stuck with: 
```java
products.stream()
    .filter(p -> p.getPrice() > 100 
        && p.getStock() > 0 
        && p.isPremium() 
        && p.isSeasonal())
    .collect(Collectors.toList());
``` 
Gets messy quickly!

**WHAT'S THE PROBLEM?**  
- Complex filtering logic scattered everywhere
- Hard to combine multiple criteria
- Can't reuse business rules
- Testing becomes difficult

**HOW SPECIFICATION SAVES YOU**  
Create separate objects for business rules.

You say: "Here's a specification that defines the rules." 

And the Specification:  
- Encapsulates business rules
- Combines rules using AND, OR, NOT
- Makes rules reusable
- Easy to test

Now you just: 
```java
Specification<Product> spec = new PremiumSpec()
    .and(new InStockSpec())
    .or(new SeasonalSpec());
products = productRepository.findAll(spec);
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
interface Specification<T> {
    boolean isSatisfiedBy(T item);
    
    default Specification<T> and(Specification<T> other) {
        return item -> isSatisfiedBy(item) && other.isSatisfiedBy(item);
    }
    
    default Specification<T> or(Specification<T> other) {
        return item -> isSatisfiedBy(item) || other.isSatisfiedBy(item);
    }
    
    default Specification<T> not() {
        return item -> !isSatisfiedBy(item);
    }
}

class PriceSpecification implements Specification<Product> {
    private final double minPrice;
    
    public PriceSpecification(double minPrice) {
        this.minPrice = minPrice;
    }
    
    @Override
    public boolean isSatisfiedBy(Product product) {
        return product.getPrice() >= minPrice;
    }
}

class InStockSpecification implements Specification<Product> {
    @Override
    public boolean isSatisfiedBy(Product product) {
        return product.getStock() > 0;
    }
}

// Usage
Specification<Product> spec = new PriceSpecification(100)
    .and(new InStockSpecification());
List<Product> products = productRepository.findBySpecification(spec);
```

#### TypeScript  
```typescript  
interface Specification<T> {
    isSatisfiedBy(item: T): boolean;
    and(other: Specification<T>): Specification<T>;
    or(other: Specification<T>): Specification<T>;
    not(): Specification<T>;
}

abstract class CompositeSpecification<T> implements Specification<T> {
    abstract isSatisfiedBy(item: T): boolean;
    
    and(other: Specification<T>): Specification<T> {
        return new AndSpecification(this, other);
    }
    
    or(other: Specification<T>): Specification<T> {
        return new OrSpecification(this, other);
    }
    
    not(): Specification<T> {
        return new NotSpecification(this);
    }
}

class PriceSpecification extends CompositeSpecification<Product> {
    constructor(private minPrice: number) {
        super();
    }
    
    isSatisfiedBy(product: Product): boolean {
        return product.price >= this.minPrice;
    }
}

class InStockSpecification extends CompositeSpecification<Product> {
    isSatisfiedBy(product: Product): boolean {
        return product.stock > 0;
    }
}

// Composite specifications
class AndSpecification<T> extends CompositeSpecification<T> {
    constructor(
        private left: Specification<T>,
        private right: Specification<T>
    ) {
        super();
    }
    
    isSatisfiedBy(item: T): boolean {
        return this.left.isSatisfiedBy(item) 
            && this.right.isSatisfiedBy(item);
    }
}
```

#### Python  
```python  
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import List

class Specification(ABC):
    @abstractmethod
    def is_satisfied_by(self, item) -> bool:
        pass
    
    def __and__(self, other):
        return AndSpecification(self, other)
    
    def __or__(self, other):
        return OrSpecification(self, other)
    
    def __invert__(self):
        return NotSpecification(self)

@dataclass
class Product:
    name: str
    price: float
    stock: int

class PriceSpecification(Specification):
    def __init__(self, min_price: float):
        self.min_price = min_price
    
    def is_satisfied_by(self, product: Product) -> bool:
        return product.price >= self.min_price

class InStockSpecification(Specification):
    def is_satisfied_by(self, product: Product) -> bool:
        return product.stock > 0

class AndSpecification(Specification):
    def __init__(self, *specifications):
        self.specifications = specifications
    
    def is_satisfied_by(self, item) -> bool:
        return all(
            spec.is_satisfied_by(item) 
            for spec in self.specifications
        )

# Usage
spec = PriceSpecification(100) & InStockSpecification()
products = [p for p in all_products if spec.is_satisfied_by(p)]
```

#### Go  
```go  
package main

// Specification interface
type Specification interface {
    IsSatisfiedBy(product Product) bool
}

// Product struct
type Product struct {
    Name  string
    Price float64
    Stock int
}

// PriceSpecification
type PriceSpecification struct {
    minPrice float64
}

func (s PriceSpecification) IsSatisfiedBy(product Product) bool {
    return product.Price >= s.minPrice
}

// InStockSpecification
type InStockSpecification struct{}

func (s InStockSpecification) IsSatisfiedBy(product Product) bool {
    return product.Stock > 0
}

// AndSpecification
type AndSpecification struct {
    specs []Specification
}

func (s AndSpecification) IsSatisfiedBy(product Product) bool {
    for _, spec := range s.specs {
        if !spec.IsSatisfiedBy(product) {
            return false
        }
    }
    return true
}

// Specification builder
func And(specs ...Specification) Specification {
    return AndSpecification{specs}
}

// Usage
func FindProducts(products []Product, spec Specification) []Product {
    var result []Product
    for _, product := range products {
        if spec.IsSatisfiedBy(product) {
            result = append(result, product)
        }
    }
    return result
}
```

**Key Differences:**  
- Java uses default interface methods for combinations
- TypeScript implements composite pattern for specifications
- Python uses operator overloading for natural syntax
- Go uses composition for specification combinations

**WHEN SHOULD YOU USE IT?**  
- When you have complex business rules
- When you need to combine multiple criteria
- When business rules need to be reusable
- When you want to keep domain rules separate from infrastructure

**WHERE YOU'VE ALREADY SEEN IT**  
- Database query builders
- Search filters
- Business rule engines
- Domain-driven design repositories
