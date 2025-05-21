# Design Patterns: Structural - Adapter

## Adapter Pattern - Simple Explanation

**SITUATION: YOU’RE INTEGRATING INCOMPATIBLE INTERFACES**  
You’re building an app that uses a new `PaymentGateway`. But your app expects an old interface: `OldPaymentSystem`. Initially, you try: 

#### Java
```java
OldPaymentSystem payment = new OldPaymentSystem();
``` 
But the new gateway doesn’t match.

**THEN THE MISMATCH GROWS**  
The new `PaymentGateway` has: 

```java
charge(amount)
``` 
But your app expects: 
```java
processPayment(amount)
``` 
You’re stuck rewriting your app or the gateway to match interfaces: 
```java
// Old system expects payment.processPayment(amount); 
// New system provides gateway.charge(amount);
```

**WHAT’S THE PROBLEM?**  
- The two systems have incompatible interfaces.  
- Rewriting either system is time-consuming and risky.  
- You can’t reuse the new gateway without major changes.  
- It’s a maintenance nightmare.

**HOW ADAPTER SAVES YOU**  
Instead of rewriting, you create a `PaymentAdapter`. 

You say: “Hey Adapter, make the new gateway work with my old system.” 

And the Adapter:  
- Wraps the new `PaymentGateway`.  
- Translates `processPayment` to `charge`.  
- Lets your app use the new gateway seamlessly.  
Now you just: 

```java
OldPaymentSystem payment = new PaymentAdapter(new PaymentGateway());
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
interface OldPaymentSystem {  
    void processPayment(double amount);  
}  
class PaymentGateway {  
    public void charge(double amount) {  
        System.out.println("Charging " + amount);  
    }  
}  
class PaymentAdapter implements OldPaymentSystem {  
    private PaymentGateway gateway;  
    public PaymentAdapter(PaymentGateway gateway) {  
        this.gateway = gateway;  
    }  
    public void processPayment(double amount) {  
        gateway.charge(amount);  
    }  
}  
// Usage  
PaymentGateway gateway = new PaymentGateway();  
OldPaymentSystem payment = new PaymentAdapter(gateway);  
payment.processPayment(100);  
```

#### TypeScript  
```typescript  
interface OldPaymentSystem {  
    processPayment(amount: number): void;  
}  
class PaymentGateway {  
    charge(amount: number) {  
        console.log(`Charging ${amount}`);  
    }  
}  
class PaymentAdapter implements OldPaymentSystem {  
    private gateway: PaymentGateway;  
    constructor(gateway: PaymentGateway) {  
        this.gateway = gateway;  
    }  
    processPayment(amount: number) {  
        this.gateway.charge(amount);  
    }  
}  
// Usage  
const gateway = new PaymentGateway();  
const payment = new PaymentAdapter(gateway);  
payment.processPayment(100);  
```

#### Python  
```python  
class OldPaymentSystem:  
    def process_payment(self, amount):  
        raise NotImplementedError  
class PaymentGateway:  
    def charge(self, amount):  
        print(f"Charging {amount}")  
class PaymentAdapter(OldPaymentSystem):  
    def __init__(self, gateway):  
        self.gateway = gateway  
    def process_payment(self, amount):  
        self.gateway.charge(amount)  
# Usage  
gateway = PaymentGateway()  
payment = PaymentAdapter(gateway)  
payment.process_payment(100)  
```

#### Go  
```go  
package main  
import "fmt"  
type OldPaymentSystem interface {  
    ProcessPayment(amount float64)  
}  
type PaymentGateway struct{}  
func (g *PaymentGateway) Charge(amount float64) {  
    fmt.Printf("Charging %f\n", amount)  
}  
type PaymentAdapter struct {  
    gateway *PaymentGateway  
}  
func (a *PaymentAdapter) ProcessPayment(amount float64) {  
    a.gateway.Charge(amount)  
}  
// Usage  
func main() {  
    gateway := &PaymentGateway{}  
    payment := &PaymentAdapter{gateway}  
    payment.ProcessPayment(100)  
}  
```

**Key Differences:**  
- Java and TypeScript use interfaces for type safety and clear contracts.  
- Python uses duck typing, making the adapter simpler but less strict.  
- Go uses interfaces without inheritance, focusing on behavior over type hierarchy.  
- All bridge incompatible interfaces seamlessly.

**WHEN SHOULD YOU USE IT?**  
- When you need to integrate incompatible interfaces.  
- When you want to reuse existing classes without modifying them.  
- When working with third-party libraries that don’t match your system.

**WHERE YOU’VE ALREADY SEEN IT**  
`Arrays.asList()` in Java (adapts arrays to lists).  
`InputStreamReader` in Java (adapts streams to readers).