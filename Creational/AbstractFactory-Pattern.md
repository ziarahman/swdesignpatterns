# Design Patterns: Creational - Abstract Factory

## Abstract Factory Pattern - Simple Explanation

**SITUATION: YOU’RE CREATING FAMILIES OF OBJECTS**  
You’re building a UI toolkit that supports multiple themes—like `DarkTheme` and `LightTheme`. Initially, you create buttons and windows directly: 

#### Java
```java
`Button button = new DarkButton(); 
Window window = new DarkWindow();
```

All good.

**THEN THE REQUIREMENTS GROW**  
Now you have:  
- `DarkTheme`: `DarkButton`, `DarkWindow`  
- `LightTheme`: `LightButton`, `LightWindow`  

You’re stuck writing: 

```java
if (theme == "dark") { 
    new DarkButton(); 
    new DarkWindow(); 
} else { 
    new LightButton(); 
    new LightWindow(); 
}
``` 
Everywhere. 

It’s repetitive.

**WHAT’S THE PROBLEM?**  
- You’re creating related objects individually, but they should be grouped by theme.  
- Adding a new theme (e.g., `NeonTheme`) means changing all creation logic.  
- Code is tightly coupled to specific classes.  
- It’s hard to ensure consistency across families of objects.

**HOW ABSTRACT FACTORY SAVES YOU**  
Instead of creating objects directly, you use an `AbstractFactory` like `ThemeFactory`. You say: “Hey Factory, give me the objects for `DarkTheme`.” And the Factory:  
- Creates a whole family of related objects (`DarkButton`, `DarkWindow`).  
- Hides the creation logic for each theme.  
- Keeps your code consistent and decoupled.  

Now you just ask: 
```java
ThemeFactory factory = new DarkThemeFactory(); 
Button button = factory.createButton();
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
interface Button {  
    void render();  
}  
interface Window {  
    void display();  
}  
interface ThemeFactory {  
    Button createButton();  
    Window createWindow();  
}  
class DarkButton implements Button {  
    public void render() {  
        System.out.println("Dark Button");  
    }  
}  
class DarkWindow implements Window {  
    public void display() {  
        System.out.println("Dark Window");  
    }  
}  
class LightButton implements Button {  
    public void render() {  
        System.out.println("Light Button");  
    }  
}  
class LightWindow implements Window {  
    public void display() {  
        System.out.println("Light Window");  
    }  
}  
class DarkThemeFactory implements ThemeFactory {  
    public Button createButton() {  
        return new DarkButton();  
    }  
    public Window createWindow() {  
        return new DarkWindow();  
    }  
}  
class LightThemeFactory implements ThemeFactory {  
    public Button createButton() {  
        return new LightButton();  
    }  
    public Window createWindow() {  
        return new LightWindow();  
    }  
}  
// Usage  
ThemeFactory factory = new DarkThemeFactory();  
Button button = factory.createButton();  
button.render();  
```

#### TypeScript  
```typescript  
interface Button {  
    render(): void;  
}  
interface Window {  
    display(): void;  
}  
interface ThemeFactory {  
    createButton(): Button;  
    createWindow(): Window;  
}  
class DarkButton implements Button {  
    render() {  
        console.log("Dark Button");  
    }  
}  
class DarkWindow implements Window {  
    display() {  
        console.log("Dark Window");  
    }  
}  
class LightButton implements Button {  
    render() {  
        console.log("Light Button");  
    }  
}  
class LightWindow implements Window {  
    display() {  
        console.log("Light Window");  
    }  
}  
class DarkThemeFactory implements ThemeFactory {  
    createButton() {  
        return new DarkButton();  
    }  
    createWindow() {  
        return new DarkWindow();  
    }  
}  
class LightThemeFactory implements ThemeFactory {  
    createButton() {  
        return new LightButton();  
    }  
    createWindow() {  
        return new LightWindow();  
    }  
}  
// Usage  
const factory: ThemeFactory = new DarkThemeFactory();  
const button = factory.createButton();  
button.render();  
```

#### Python  
```python  
from abc import ABC, abstractmethod  
class Button(ABC):  
    @abstractmethod  
    def render(self):  
        pass  
class Window(ABC):  
    @abstractmethod  
    def display(self):  
        pass  
class DarkButton(Button):  
    def render(self):  
        print("Dark Button")  
class DarkWindow(Window):  
    def display(self):  
        print("Dark Window")  
class LightButton(Button):  
    def render(self):  
        print("Light Button")  
class LightWindow(Window):  
    def display(self):  
        print("Light Window")  
class ThemeFactory(ABC):  
    @abstractmethod  
    def create_button(self):  
        pass  
    @abstractmethod  
    def create_window(self):  
        pass  
class DarkThemeFactory(ThemeFactory):  
    def create_button(self):  
        return DarkButton()  
    def create_window(self):  
        return DarkWindow()  
class LightThemeFactory(ThemeFactory):  
    def create_button(self):  
        return LightButton()  
    def create_window(self):  
        return LightWindow()  
# Usage  
factory = DarkThemeFactory()  
button = factory.create_button()  
button.render()  
```

#### Go  
```go  
package main  
import "fmt"  
type Button interface {  
    Render()  
}  
type Window interface {  
    Display()  
}  
type DarkButton struct{}  
func (b *DarkButton) Render() {  
    fmt.Println("Dark Button")  
}  
type DarkWindow struct{}  
func (w *DarkWindow) Display() {  
    fmt.Println("Dark Window")  
}  
type LightButton struct{}  
func (b *LightButton) Render() {  
    fmt.Println("Light Button")  
}  
type LightWindow struct{}  
func (w *LightWindow) Display() {  
    fmt.Println("Light Window")  
}  
type ThemeFactory interface {  
    CreateButton() Button  
    CreateWindow() Window  
}  
type DarkThemeFactory struct{}  
func (f *DarkThemeFactory) CreateButton() Button {  
    return &DarkButton{}  
}  
func (f *DarkThemeFactory) CreateWindow() Window {  
    return &DarkWindow{}  
}  
type LightThemeFactory struct{}  
func (f *LightThemeFactory) CreateButton() Button {  
    return &LightButton{}  
}  
func (f *LightThemeFactory) CreateWindow() Window {  
    return &LightWindow{}  
}  
// Usage  
func main() {  
    factory := &DarkThemeFactory{}  
    button := factory.CreateButton()  
    button.Render()  
}  
```

**Key Differences:**  
- Java and TypeScript use interfaces and inheritance for factories, ensuring type safety.  
- Python uses abstract base classes for a more dynamic approach.  
- Go relies on interfaces without inheritance, focusing on composition.  
- All ensure consistent creation of related objects.

**WHEN SHOULD YOU USE IT?**  
- When you need to create families of related objects.  
- When you want to ensure objects work together consistently.  
- When you might add new families (themes, styles) in the future.

**WHERE YOU’VE ALREADY SEEN IT**  
`DocumentBuilderFactory` in Java (for XML parsing).  
`Toolkit` in Java AWT (for platform-specific UI components).