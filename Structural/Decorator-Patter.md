# Design Patterns: Structural - Decorator

## Decorator Pattern - Simple Explanation

**SITUATION: YOU’RE ADDING FEATURES TO OBJECTS**  
You’re building a coffee shop app with a `Coffee` class. 

Initially, you have a simple coffee: 

#### Java
```java
Coffee coffee = new SimpleCoffee();
``` 

All good.

**THEN THE MENU GROWS**  
Now customers want:  
- `CoffeeWithMilk`  
- `CoffeeWithSugar`  
- `CoffeeWithMilkAndSugar`  
You’re stuck creating subclasses: 

```java
class CoffeeWithMilk extends Coffee
``` 
and 

```java
class CoffeeWithMilkAndSugar extends Coffee
``` 
Everywhere. It’s a mess.

**WHAT’S THE PROBLEM?**  
- You end up with a subclass explosion for every combination.  
- Adding a new feature (e.g., `Caramel`) means more subclasses.  
- It’s inflexible and hard to maintain.  
- You can’t add features dynamically at runtime.

**HOW DECORATOR SAVES YOU**  
Instead of subclasses, you create `CoffeeDecorator`s like `MilkDecorator`, `SugarDecorator`. 

You say: “Hey Decorator, add milk to my coffee.” 

And the Decorator:  
- Wraps the original coffee.  
- Adds the new feature (milk, sugar) dynamically.  
- Keeps your code flexible and clean.  
Now you just: 

```java
Coffee coffee = new MilkDecorator(new SugarDecorator(new SimpleCoffee()));
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
interface Coffee {  
    int cost();  
    String description();  
}  
class SimpleCoffee implements Coffee {  
    public int cost() {  
        return 5;  
    }  
    public String description() {  
        return "Simple Coffee";  
    }  
}  
abstract class CoffeeDecorator implements Coffee {  
    protected Coffee coffee;  
    public CoffeeDecorator(Coffee coffee) {  
        this.coffee = coffee;  
    }  
}  
class MilkDecorator extends CoffeeDecorator {  
    public MilkDecorator(Coffee coffee) {  
        super(coffee);  
    }  
    public int cost() {  
        return coffee.cost() + 2;  
    }  
    public String description() {  
        return coffee.description() + ", Milk";  
    }  
}  
class SugarDecorator extends CoffeeDecorator {  
    public SugarDecorator(Coffee coffee) {  
        super(coffee);  
    }  
    public int cost() {  
        return coffee.cost() + 1;  
    }  
    public String description() {  
        return coffee.description() + ", Sugar";  
    }  
}  
// Usage  
Coffee coffee = new MilkDecorator(new SugarDecorator(new SimpleCoffee()));  
System.out.println(coffee.description() + " " + coffee.cost());  
```

#### TypeScript  
```typescript  
interface Coffee {  
    cost(): number;  
    description(): string;  
}  
class SimpleCoffee implements Coffee {  
    cost() {  
        return 5;  
    }  
    description() {  
        return "Simple Coffee";  
    }  
}  
abstract class CoffeeDecorator implements Coffee {  
    constructor(protected coffee: Coffee) {}  
    abstract cost(): number;  
    abstract description(): string;  
}  
class MilkDecorator extends CoffeeDecorator {  
    cost() {  
        return this.coffee.cost() + 2;  
    }  
    description() {  
        return this.coffee.description() + ", Milk";  
    }  
}  
class SugarDecorator extends CoffeeDecorator {  
    cost() {  
        return this.coffee.cost() + 1;  
    }  
    description() {  
        return this.coffee.description() + ", Sugar";  
    }  
}  
// Usage  
const coffee: Coffee = new MilkDecorator(new SugarDecorator(new SimpleCoffee()));  
console.log(coffee.description(), coffee.cost());  
```

#### Python  
```python  
from abc import ABC, abstractmethod  
class Coffee(ABC):  
    @abstractmethod  
    def cost(self):  
        pass  
    @abstractmethod  
    def description(self):  
        pass  
class SimpleCoffee(Coffee):  
    def cost(self):  
        return 5  
    def description(self):  
        return "Simple Coffee"  
class CoffeeDecorator(Coffee):  
    def __init__(self, coffee):  
        self.coffee = coffee  
class MilkDecorator(CoffeeDecorator):  
    def cost(self):  
        return self.coffee.cost() + 2  
    def description(self):  
        return self.coffee.description() + ", Milk"  
class SugarDecorator(CoffeeDecorator):  
    def cost(self):  
        return self.coffee.cost() + 1  
    def description(self):  
        return self.coffee.description() + ", Sugar"  
# Usage  
coffee = MilkDecorator(SugarDecorator(SimpleCoffee()))  
print(coffee.description(), coffee.cost())  
```

#### Go  
```go  
package main  
import "fmt"  
type Coffee interface {  
    Cost() int  
    Description() string  
}  
type SimpleCoffee struct{}  
func (c *SimpleCoffee) Cost() int {  
    return 5  
}  
func (c *SimpleCoffee) Description() string {  
    return "Simple Coffee"  
}  
type CoffeeDecorator struct {  
    coffee Coffee  
}  
type MilkDecorator struct {  
    CoffeeDecorator  
}  
func (d *MilkDecorator) Cost() int {  
    return d.coffee.Cost() + 2  
}  
func (d *MilkDecorator) Description() string {  
    return d.coffee.Description() + ", Milk"  
}  
type SugarDecorator struct {  
    CoffeeDecorator  
}  
func (d *SugarDecorator) Cost() int {  
    return d.coffee.Cost() + 1  
}  
func (d *SugarDecorator) Description() string {  
    return d.coffee.Description() + ", Sugar"  
}  
// Usage  
func main() {  
    coffee := &SugarDecorator{&MilkDecorator{&SimpleCoffee{}}}  
    fmt.Println(coffee.Description(), coffee.Cost())  
}  
```

**Key Differences:**  
- Java and TypeScript use inheritance and interfaces for a structured approach.  
- Python uses duck typing, making decorators simpler but less strict.  
- Go uses embedded structs to achieve composition, avoiding inheritance.  
- All enable dynamic addition of features.

**WHEN SHOULD YOU USE IT?**  
- When you need to add features to objects dynamically.  
- When subclassing leads to too many combinations.  
- When you want to extend behavior without modifying the original class.

**WHERE YOU’VE ALREADY SEEN IT**  
`BufferedReader` in Java (wraps a reader to add buffering).  
`ScrollPane` in GUI frameworks (adds scrolling to components).