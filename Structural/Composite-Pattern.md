# Design Patterns: Structural - Composite

## Composite Pattern - Simple Explanation

**SITUATION: YOU’RE WORKING WITH HIERARCHICAL STRUCTURES**  
You’re building a file system with `Files` and `Folders`. 

Initially, you treat them differently: 

#### Java
```java
file.read(); 
folder.listFiles();
``` 
All good.

**THEN THE SYSTEM GROWS**  
Now you need to perform operations like `getSize()` on both files and folders. 

You’re stuck writing: 
```java
if (item instanceof File) { 
    file.getSize(); 
} else { 
    folder.getSize(); 
}
``` 
Everywhere. 

It’s repetitive and error-prone.

**WHAT’S THE PROBLEM?**  
- You’re treating individual objects (`File`) and collections (`Folder`) differently.  
- Adding new operations means updating all conditional logic.  
- It violates the Open/Closed Principle.  
- The code becomes hard to maintain as the hierarchy grows.

**HOW COMPOSITE SAVES YOU**  
Instead of treating them differently, you create a common interface `Component` with methods like `getSize()`. Both `File` and `Folder` implement `Component`. A `Folder` can contain other `Component`s (files or subfolders). 

You say: “Hey Component, give me your size,” and it works for both files and folders. 

Now you just: 
```java
Component file = new File(); 
Component folder = new Folder(); 
folder.add(file); 
folder.getSize();
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
interface Component {  
    int getSize();  
    void add(Component component);  
}  
class File implements Component {  
    private int size;  
    public File(int size) {  
        this.size = size;  
    }  
    public int getSize() {  
        return size;  
    }  
    public void add(Component component) {  
        throw new UnsupportedOperationException();  
    }  
}  
class Folder implements Component {  
    private List<Component> components = new ArrayList<>();  
    public void add(Component component) {  
        components.add(component);  
    }  
    public int getSize() {  
        return components.stream().mapToInt(Component::getSize).sum();  
    }  
}  
// Usage  
Component file = new File(100);  
Component folder = new Folder();  
folder.add(file);  
System.out.println(folder.getSize());  
```

#### TypeScript  
```typescript  
interface Component {  
    getSize(): number;  
    add?(component: Component): void;  
}  
class File implements Component {  
    constructor(private size: number) {}  
    getSize() {  
        return this.size;  
    }  
}  
class Folder implements Component {  
    private components: Component[] = [];  
    add(component: Component) {  
        this.components.push(component);  
    }  
    getSize() {  
        return this.components.reduce((total, c) => total + c.getSize(), 0);  
    }  
}  
// Usage  
const file = new File(100);  
const folder = new Folder();  
folder.add(file);  
console.log(folder.getSize());  
```

#### Python  
```python  
from abc import ABC, abstractmethod  
class Component(ABC):  
    @abstractmethod  
    def get_size(self):  
        pass  
class File(Component):  
    def __init__(self, size):  
        self.size = size  
    def get_size(self):  
        return self.size  
class Folder(Component):  
    def __init__(self):  
        self.components = []  
    def add(self, component):  
        self.components.append(component)  
    def get_size(self):  
        return sum(c.get_size() for c in self.components)  
# Usage  
file = File(100)  
folder = Folder()  
folder.add(file)  
print(folder.get_size())  
```

#### Go  
```go  
package main  
import "fmt"  
type Component interface {  
    GetSize() int  
}  
type File struct {  
    size int  
}  
func (f *File) GetSize() int {  
    return f.size  
}  
type Folder struct {  
    components []Component  
}  
func (f *Folder) Add(component Component) {  
    f.components = append(f.components, component)  
}  
func (f *Folder) GetSize() int {  
    total := 0  
    for _, c := range f.components {  
        total += c.GetSize()  
    }  
    return total  
}  
// Usage  
func main() {  
    file := &File{100}  
    folder := &Folder{}  
    folder.Add(file)  
    fmt.Println(folder.GetSize())  
}  
```

**Key Differences:**  
- Java and TypeScript use interfaces for a uniform component contract.  
- Python uses abstract base classes for flexibility.  
- Go uses interfaces and slices for composition, avoiding inheritance.  
- All treat hierarchies uniformly.

**WHEN SHOULD YOU USE IT?**  
- When you have a part-whole hierarchy (e.g., files and folders, UI components).  
- When you want to treat individual objects and compositions the same way.  
- When you need to perform operations recursively on a tree structure.

**WHERE YOU’VE ALREADY SEEN IT**  
`DOM` in HTML (elements and their children).  
`JPanel` in Java Swing (can contain other components).