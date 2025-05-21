# Design Patterns: Behavioral - Oberserver

## Observer Pattern - Simple Explanation

**SITUATION: YOU’RE HANDLING EVENT NOTIFICATIONS**  
You’re building a stock app where `StockPrice` updates need to notify displays. 

Initially, you hardcode: 

#### Java
```java
display.update(stockPrice);
``` 
All good.

**THEN THE APP GROWS**  
Now you have:  
- `MobileDisplay`  
- `WebDisplay`  
- `EmailAlert`  

You’re stuck calling: 
```java
mobileDisplay.update(); 
webDisplay.update(); 
emailAlert.send();
``` 
Everywhere. It’s repetitive.

**WHAT’S THE PROBLEM?**  
- The `StockPrice` is tightly coupled to all displays.  
- Adding a new display means changing the `StockPrice` class.  
- You’re manually managing who gets updates.  
- It’s hard to scale and maintain.

**HOW OBSERVER SAVES YOU**  
Instead of hardcoding, you make `StockPrice` a subject and displays its observers. 

You say: “Hey Observers, `StockPrice` changed—update yourselves.” 

And the Observer:  
- Lets `StockPrice` notify all observers automatically.  
- Decouples the subject from its observers.  
- Makes adding new observers easy.  

Now you just: 
```java
stockPrice.addObserver(mobileDisplay); 
stockPrice.setPrice(100);
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
interface Observer {  
    void update(double price);  
}  
interface Subject {  
    void addObserver(Observer observer);  
    void removeObserver(Observer observer);  
    void notifyObservers();  
}  
class StockPrice implements Subject {  
    private List<Observer> observers = new ArrayList<>();  
    private double price;  
    public void addObserver(Observer observer) {  
        observers.add(observer);  
    }  
    public void removeObserver(Observer observer) {  
        observers.remove(observer);  
    }  
    public void notifyObservers() {  
        for (Observer observer : observers) observer.update(price);  
    }  
    public void setPrice(double price) {  
        this.price = price;  
        notifyObservers();  
    }  
}  
class MobileDisplay implements Observer {  
    public void update(double price) {  
        System.out.println("Mobile: Price updated to " + price);  
    }  
}  
// Usage  
StockPrice stock = new StockPrice();  
Observer display = new MobileDisplay();  
stock.addObserver(display);  
stock.setPrice(100);  
```

#### TypeScript  
```typescript  
interface Observer {  
    update(price: number): void;  
}  
interface Subject {  
    addObserver(observer: Observer): void;  
    removeObserver(observer: Observer): void;  
    notifyObservers(): void;  
}  
class StockPrice implements Subject {  
    private observers: Observer[] = [];  
    private price: number = 0;  
    addObserver(observer: Observer) {  
        this.observers.push(observer);  
    }  
    removeObserver(observer: Observer) {  
        this.observers = this.observers.filter(obs => obs !== observer);  
    }  
    notifyObservers() {  
        this.observers.forEach(obs => obs.update(this.price));  
    }  
    setPrice(price: number) {  
        this.price = price;  
        this.notifyObservers();  
    }  
}  
class MobileDisplay implements Observer {  
    update(price: number) {  
        console.log(`Mobile: Price updated to ${price}`);  
    }  
}  
// Usage  
const stock = new StockPrice();  
const display = new MobileDisplay();  
stock.addObserver(display);  
stock.setPrice(100);  
```

#### Python  
``python  
class Observer:  
    def update(self, price):  
        raise NotImplementedError  
class Subject:  
    def add_observer(self, observer):  
        raise NotImplementedError  
    def remove_observer(self, observer):  
        raise NotImplementedError  
    def notify_observers(self):  
        raise NotImplementedError  
class StockPrice(Subject):  
    def __init__(self):  
        self.observers = []  
        self.price = 0  
    def add_observer(self, observer):  
        self.observers.append(observer)  
    def remove_observer(self, observer):  
        self.observers.remove(observer)  
    def notify_observers(self):  
        for observer in self.observers:  
            observer.update(self.price)  
    def set_price(self, price):  
        self.price = price  
        self.notify_observers()  
class MobileDisplay(Observer):  
    def update(self, price):  
        print(f"Mobile: Price updated to {price}")  
# Usage  
stock = StockPrice()  
display = MobileDisplay()  
stock.add_observer(display)  
stock.set_price(100)  
```

#### Go  
```go  
package main  
import "fmt"  
type Observer interface {  
    Update(price float64)  
}  
type Subject interface {  
    AddObserver(observer Observer)  
    RemoveObserver(observer Observer)  
    NotifyObservers()  
}  
type StockPrice struct {  
    observers []Observer  
    price float64  
}  
func (s *StockPrice) AddObserver(observer Observer) {  
    s.observers = append(s.observers, observer)  
}  
func (s *StockPrice) RemoveObserver(observer Observer) {  
    for i, obs := range s.observers {  
        if obs == observer {  
            s.observers = append(s.observers[:i], s.observers[i+1:]...)  
            break  
        }  
    }  
}  
func (s *StockPrice) NotifyObservers() {  
    for _, observer := range s.observers {  
        observer.Update(s.price)  
    }  
}  
func (s *StockPrice) SetPrice(price float64) {  
    s.price = price  
    s.NotifyObservers()  
}  
type MobileDisplay struct{}  
func (d *MobileDisplay) Update(price float64) {  
    fmt.Printf("Mobile: Price updated to %f\n", price)  
}  
// Usage  
func main() {  
    stock := &StockPrice{}  
    display := &MobileDisplay{}  
    stock.AddObserver(display)  
    stock.SetPrice(100)  
}  
```

**Key Differences:**  
- Java and TypeScript use interfaces for a structured observer-subject relationship.  
- Python uses a more dynamic approach with lists and method overriding.  
- Go uses slices for observers and interfaces for behavior, avoiding inheritance.  
- All enable decoupled event notifications.

**WHEN SHOULD YOU USE IT?**  
- When one object’s change needs to notify others.  
- When you want to decouple the notifier from the receivers.  
- When the list of observers might change dynamically.

**WHERE YOU’VE ALREADY SEEN IT**  
`addActionListener` in Java GUI (for event handling).  
`Redux` in JavaScript (for state changes).