# Design Patterns: Creational - Builder

## Builder Pattern - Simple Explanations

**SITUATION: YOU’RE CREATING COMPLEX OBJECTS**  
You’re building a feature that creates a `Car` object with many optional parts. Initially, you have a simple constructor: 

#### Java
```java
Car car = new Car("Toyota", "Camry");
```
All good.

**THEN THE REQUIREMENTS GROW**  
Now your `Car` has:  
- Optional sunroof.  
- Optional leather seats.  
- Optional navigation system.  
- Optional color.  

You’re stuck with a giant constructor: 

#### Java
```java
Car car = new Car("Toyota", "Camry", true, false, true, "red");
```
Or worse, multiple constructors: 
```java
Car(String make, String model) 
```
and 
```java
Car(String make, String model, boolean sunroof, boolean seats)
``` 
Everywhere. It’s a mess.

**WHAT’S THE PROBLEM?**  
- Constructors with too many parameters are hard to read.  
- You’re forced to pass `null` or default values for optional fields.  
- Adding new options means changing the constructor—breaking existing code.  
- It’s error-prone and confusing for developers.

**HOW BUILDER SAVES YOU**  
Instead of using constructors directly, you create a `CarBuilder`. You say: “Hey Builder, let me build a `Car` step by step.” And the Builder:  
- Lets you set properties fluently and optionally.  
- Keeps the construction process clear and readable.  
- Creates the final object only when you’re ready.  
Now you build it like: `Car car = new CarBuilder("Toyota", "Camry").withSunroof(true).withColor("red").build();`

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
class Car {  
    private String make, model, color;  
    private boolean sunroof, leatherSeats, navigation;  
    public Car(String make, String model, boolean sunroof, boolean leatherSeats, boolean navigation, String color) {  
        this.make = make;  
        this.model = model;  
        this.sunroof = sunroof;  
        this.leatherSeats = leatherSeats;  
        this.navigation = navigation;  
        this.color = color;  
    }  
}  
class CarBuilder {  
    private String make, model, color;  
    private boolean sunroof, leatherSeats, navigation;  
    public CarBuilder(String make, String model) {  
        this.make = make;  
        this.model = model;  
    }  
    public CarBuilder withSunroof(boolean sunroof) {  
        this.sunroof = sunroof;  
        return this;  
    }  
    public CarBuilder withLeatherSeats(boolean seats) {  
        this.leatherSeats = seats;  
        return this;  
    }  
    public CarBuilder withNavigation(boolean nav) {  
        this.navigation = nav;  
        return this;  
    }  
    public CarBuilder withColor(String color) {  
        this.color = color;  
        return this;  
    }  
    public Car build() {  
        return new Car(make, model, sunroof, leatherSeats, navigation, color);  
    }  
}  
// Usage  
Car car = new CarBuilder("Toyota", "Camry").withSunroof(true).withColor("red").build();  
```

#### TypeScript  
```typescript  
class Car {  
    constructor(  
        public make: string,  
        public model: string,  
        public sunroof?: boolean,  
        public leatherSeats?: boolean,  
        public navigation?: boolean,  
        public color?: string  
    ) {}  
}  
class CarBuilder {  
    private make: string;  
    private model: string;  
    private sunroof?: boolean;  
    private leatherSeats?: boolean;  
    private navigation?: boolean;  
    private color?: string;  
    constructor(make: string, model: string) {  
        this.make = make;  
        this.model = model;  
    }  
    withSunroof(sunroof: boolean) {  
        this.sunroof = sunroof;  
        return this;  
    }  
    withLeatherSeats(seats: boolean) {  
        this.leatherSeats = seats;  
        return this;  
    }  
    withNavigation(nav: boolean) {  
        this.navigation = nav;  
        return this;  
    }  
    withColor(color: string) {  
        this.color = color;  
        return this;  
    }  
    build(): Car {  
        return new Car(this.make, this.model, this.sunroof, this.leatherSeats, this.navigation, this.color);  
    }  
}  
// Usage  
const car = new CarBuilder("Toyota", "Camry").withSunroof(true).withColor("red").build();  
```

#### Python  
```python  
class Car:  
    def __init__(self, make, model, sunroof=None, leather_seats=None, navigation=None, color=None):  
        self.make, self.model = make, model  
        self.sunroof, self.leather_seats, self.navigation, self.color = sunroof, leather_seats, navigation, color  
class CarBuilder:  
    def __init__(self, make, model):  
        self.car = Car(make, model)  
    def with_sunroof(self, sunroof):  
        self.car.sunroof = sunroof  
        return self  
    def with_leather_seats(self, seats):  
        self.car.leather_seats = seats  
        return self  
    def with_navigation(self, nav):  
        self.car.navigation = nav  
        return self  
    def with_color(self, color):  
        self.car.color = color  
        return self  
    def build(self):  
        return self.car  
# Usage  
car = CarBuilder("Toyota", "Camry").with_sunroof(True).with_color("red").build()  
```

#### Go  
```go  
package main  
type Car struct {  
    Make, Model string  
    Sunroof, LeatherSeats, Navigation bool  
    Color string  
}  
type CarBuilder struct {  
    car Car  
}  
func NewCarBuilder(make, model string) *CarBuilder {  
    return &CarBuilder{Car{make, model, false, false, false, ""}}  
}  
func (b *CarBuilder) WithSunroof(sunroof bool) *CarBuilder {  
    b.car.Sunroof = sunroof  
    return b  
}  
func (b *CarBuilder) WithLeatherSeats(seats bool) *CarBuilder {  
    b.car.LeatherSeats = seats  
    return b  
}  
func (b *CarBuilder) WithNavigation(nav bool) *CarBuilder {  
    b.car.Navigation = nav  
    return b  
}  
func (b *CarBuilder) WithColor(color string) *CarBuilder {  
    b.car.Color = color  
    return b  
}  
func (b *CarBuilder) Build() Car {  
    return b.car  
}  
// Usage  
func main() {  
    car := NewCarBuilder("Toyota", "Camry").WithSunroof(true).WithColor("red").Build()  
}  
```

**Key Differences:**  
- Java and TypeScript use method chaining for a fluent API, emphasizing type safety.  
- Python uses a more flexible approach, often with optional parameters and dynamic attributes.  
- Go avoids method receivers for the fluent API but achieves the same result with pointer returns.  
- All provide a clear, step-by-step construction process.

**WHEN SHOULD YOU USE IT?**  
- When your object has many optional parameters or complex setup.  
- When you want a clear, step-by-step construction process.  
- When you need to ensure the object is immutable after creation.

**WHERE YOU’VE ALREADY SEEN IT**  
`StringBuilder` in Java (for building strings).  
`AlertDialog.Builder` in Android (for building dialogs).  
`Request.Builder` in OkHttp (for building HTTP requests).