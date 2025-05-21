# Design Patterns: Structural - Bridge

## Bridge Pattern - Simple Explnation

**SITUATION: YOU’RE DEALING WITH MULTIPLE DIMENSIONS OF VARIATION**  
You’re building a drawing app with shapes like `Circle` and `Square`, and they need to be rendered on different platforms like `Windows` and `Mac`. 

Initially, you create classes like: 

#### Java
```java
WindowsCircle circle = new WindowsCircle(); 
MacSquare square = new MacSquare();
``` 
All good.

**THEN THE APP GROWS**  
Now you add more shapes (`Triangle`) and platforms (`Linux`). 

You’re stuck creating: 
```java
WindowsTriangle triangle = new WindowsTriangle(); 
MacTriangle macTriangle = new MacTriangle(); 
LinuxTriangle linuxTriangle = new LinuxTriangle();
``` 
The number of classes explodes.

**WHAT’S THE PROBLEM?**  
- You’re creating a new class for every shape-platform combination, leading to a class explosion.  
- Adding a new platform or shape means creating multiple new classes.  
- Code becomes tightly coupled and hard to maintain.  
- It violates the Open/Closed Principle.

**HOW BRIDGE SAVES YOU**  
Instead of creating a class for every combination, you separate the abstraction (`Shape`) from the implementation (`Renderer`). You create a `Renderer` interface with implementations like `WindowsRenderer` and `MacRenderer`. Then, `Shape` classes like `Circle` and `Square` use a `Renderer` to draw themselves. 

You say: “Hey Shape, use this renderer to draw yourself.” 

Now you just: 
```java
Shape circle = new Circle(new WindowsRenderer()); 
circle.draw();
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
interface Renderer {  
    void renderShape(String shape);  
}  
class WindowsRenderer implements Renderer {  
    public void renderShape(String shape) {  
        System.out.println("Rendering " + shape + " on Windows");  
    }  
}  
class MacRenderer implements Renderer {  
    public void renderShape(String shape) {  
        System.out.println("Rendering " + shape + " on Mac");  
    }  
}  
abstract class Shape {  
    protected Renderer renderer;  
    public Shape(Renderer renderer) {  
        this.renderer = renderer;  
    }  
    abstract void draw();  
}  
class Circle extends Shape {  
    public Circle(Renderer renderer) {  
        super(renderer);  
    }  
    public void draw() {  
        renderer.renderShape("Circle");  
    }  
}  
// Usage  
Shape circle = new Circle(new WindowsRenderer());  
circle.draw();  
```

#### TypeScript  
```typescript  
interface Renderer {  
    renderShape(shape: string): void;  
}  
class WindowsRenderer implements Renderer {  
    renderShape(shape: string) {  
        console.log(`Rendering ${shape} on Windows`);  
    }  
}  
class MacRenderer implements Renderer {  
    renderShape(shape: string) {  
        console.log(`Rendering ${shape} on Mac`);  
    }  
}  
abstract class Shape {  
    constructor(protected renderer: Renderer) {}  
    abstract draw(): void;  
}  
class Circle extends Shape {  
    draw() {  
        this.renderer.renderShape("Circle");  
    }  
}  
// Usage  
const circle = new Circle(new WindowsRenderer());  
circle.draw();  
```

#### Python  
```python  
from abc import ABC, abstractmethod  
class Renderer(ABC):  
    @abstractmethod  
    def render_shape(self, shape):  
        pass  
class WindowsRenderer(Renderer):  
    def render_shape(self, shape):  
        print(f"Rendering {shape} on Windows")  
class MacRenderer(Renderer):  
    def render_shape(self, shape):  
        print(f"Rendering {shape} on Mac")  
class Shape(ABC):  
    def __init__(self, renderer):  
        self.renderer = renderer  
    @abstractmethod  
    def draw(self):  
        pass  
class Circle(Shape):  
    def draw(self):  
        self.renderer.render_shape("Circle")  
# Usage  
circle = Circle(WindowsRenderer())  
circle.draw()  
```

#### Go  
```go  
package main  
import "fmt"  
type Renderer interface {  
    RenderShape(shape string)  
}  
type WindowsRenderer struct{}  
func (r *WindowsRenderer) RenderShape(shape string) {  
    fmt.Printf("Rendering %s on Windows\n", shape)  
}  
type MacRenderer struct{}  
func (r *MacRenderer) RenderShape(shape string) {  
    fmt.Printf("Rendering %s on Mac\n", shape)  
}  
type Shape interface {  
    Draw()  
}  
type Circle struct {  
    renderer Renderer  
}  
func (c *Circle) Draw() {  
    c.renderer.RenderShape("Circle")  
}  
// Usage  
func main() {  
    circle := &Circle{&WindowsRenderer{}}  
    circle.Draw()  
}  
```

**Key Differences:**  
- Java and TypeScript use interfaces and inheritance for abstraction and implementation separation.  
- Python uses abstract base classes for a more dynamic approach.  
- Go relies on interfaces and composition, avoiding inheritance.  
- All decouple abstraction from implementation.

**WHEN SHOULD YOU USE IT?**  
- When you have multiple dimensions of variation (e.g., shapes and platforms).  
- When you want to decouple abstraction from implementation.  
- When you need to extend both sides independently without a class explosion.

**WHERE YOU’VE ALREADY SEEN IT**  
`JDBC` drivers in Java (decouples database operations from specific database implementations).  
`Graphics` APIs (separates drawing logic from platform-specific rendering).