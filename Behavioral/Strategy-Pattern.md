# Design Patterns: Behavioral - Strategy

## Strategy Pattern - Simple Explantion

**SITUATION: YOU’RE IMPLEMENTING ALGORITHMS**  
You’re building a payment system with different methods. 

Initially, you have: 

#### Java
```java
payment.process("credit");
``` 
All good.

**THEN THE METHODS GROW**  
Now you have:  
- `CreditCardPayment`  
- `PayPalPayment`  
- `CryptoPayment`  

You’re stuck with: 
```java
if (method == "credit") { 
    creditCard.process(); 
} else if (method == "paypal") { 
    paypal.process(); 
}
``` 
Everywhere.

**WHAT’S THE PROBLEM?**  
- The payment logic is hardcoded and repetitive.  
- Adding a new method means changing the `Payment` class.  
- It violates the Open/Closed Principle.  
- Testing different methods is hard.

**HOW STRATEGY SAVES YOU**  
Instead of hardcoding, you define a `PaymentStrategy` interface and implementations. 

You say: “Hey Strategy, process this payment your way.” 

And the Strategy:  
- Encapsulates each algorithm (credit, PayPal).  
- Lets you swap strategies at runtime.  
- Keeps your code clean and extensible.  

Now you just: 
```java
Payment payment = new Payment(new CreditCardStrategy()); payment.process();
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
interface PaymentStrategy {  
    void process(double amount);  
}  
class CreditCardStrategy implements PaymentStrategy {  
    public void process(double amount) {  
        System.out.println("Processing " + amount + " via Credit Card");  
    }  
}  
class PayPalStrategy implements PaymentStrategy {  
    public void process(double amount) {  
        System.out.println("Processing " + amount + " via PayPal");  
    }  
}  
class Payment {  
    private PaymentStrategy strategy;  
    public Payment(PaymentStrategy strategy) {  
        this.strategy = strategy;  
    }  
    public void process(double amount) {  
        strategy.process(amount);  
    }  
    public void setStrategy(PaymentStrategy strategy) {  
        this.strategy = strategy;  
    }  
}  
// Usage  
Payment payment = new Payment(new CreditCardStrategy());  
payment.process(100);  
```

#### TypeScript  
```typescript  
interface PaymentStrategy {  
    process(amount: number): void;  
}  
class CreditCardStrategy implements PaymentStrategy {  
    process(amount: number) {  
        console.log(`Processing ${amount} via Credit Card`);  
    }  
}  
class PayPalStrategy implements PaymentStrategy {  
    process(amount: number) {  
        console.log(`Processing ${amount} via PayPal`);  
    }  
}  
class Payment {  
    constructor(private strategy: PaymentStrategy) {}  
    process(amount: number) {  
        this.strategy.process(amount);  
    }  
    setStrategy(strategy: PaymentStrategy) {  
        this.strategy = strategy;  
    }  
}  
// Usage  
const payment = new Payment(new CreditCardStrategy());  
payment.process(100);  
```

#### Python  
```python  
from abc import ABC, abstractmethod  
class PaymentStrategy(ABC):  
    @abstractmethod  
    def process(self, amount):  
        pass  
class CreditCardStrategy(PaymentStrategy):  
    def process(self, amount):  
        print(f"Processing {amount} via Credit Card")  
class PayPalStrategy(PaymentStrategy):  
    def process(self, amount):  
        print(f"Processing {amount} via PayPal")  
class Payment:  
    def __init__(self, strategy):  
        self.strategy = strategy  
    def process(self, amount):  
        self.strategy.process(amount)  
    def set_strategy(self, strategy):  
        self.strategy = strategy  
# Usage  
payment = Payment(CreditCardStrategy())  
payment.process(100)  
```

#### Go  
```go  
package main  
import "fmt"  
type PaymentStrategy interface {  
    Process(amount float64)  
}  
type CreditCardStrategy struct{}  
func (s *CreditCardStrategy) Process(amount float64) {  
    fmt.Printf("Processing %f via Credit Card\n", amount)  
}  
type PayPalStrategy struct{}  
func (s *PayPalStrategy) Process(amount float64) {  
    fmt.Printf("Processing %f via PayPal\n", amount)  
}  
type Payment struct {  
    strategy PaymentStrategy  
}  
func (p *Payment) Process(amount float64) {  
    p.strategy.Process(amount)  
}  
func (p *Payment) SetStrategy(strategy PaymentStrategy) {  
    p.strategy = strategy  
}  
// Usage  
func main() {  
    payment := &Payment{&CreditCardStrategy{}}  
    payment.Process(100)  
}  
```

**Key Differences:**  
- Java and TypeScript use interfaces for a clear strategy contract.  
- Python uses abstract base classes or duck typing for flexibility.  
- Go uses interfaces and composition, keeping it lightweight.  
- All allow runtime swapping of algorithms.

**WHEN SHOULD YOU USE IT?**  
- When you have multiple algorithms for a task.  
- When you want to swap algorithms at runtime.  
- When you want to avoid conditional logic for behavior.

**WHERE YOU’VE ALREADY SEEN IT**  
`Comparator` in Java (for sorting strategies).  
`TextWatcher` in Android (for text change handling).