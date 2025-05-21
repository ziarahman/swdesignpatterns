# Design Patterns: Structural - Proxy

## Proxy Pattern - Simple Explanation

**SITUATION: YOU NEED TO CONTROL ACCESS TO AN OBJECT**  
You’re building an app that loads images. 

Initially, you load them directly: 

#### Java
```java
Image image = new RealImage("photo.jpg"); 
image.display();
``` 
All good.

**THEN THE APP GROWS**  
Now you have many images, and loading them is slow and resource-heavy. 

You’re stuck with: 
```java
RealImage image = new RealImage("photo.jpg");
``` 
Which loads the image even if you don’t display it right away.

**WHAT’S THE PROBLEM?**  
- Loading the real object (`RealImage`) is expensive and unnecessary until needed.  
- You can’t control access or add extra logic (e.g., caching, permissions).  
- Performance suffers due to eager loading.  
- It’s hard to add features like lazy loading or logging.

**HOW PROXY SAVES YOU**  
Instead of loading the real image directly, you create a `ProxyImage` that acts as a stand-in. The proxy delays loading the real image until `display()` is called. 

You say: “Hey Proxy, display the image when I need it.” 

Now you just: 

```java
Image image = new ProxyImage("photo.jpg"); 
image.display();
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
interface Image {  
    void display();  
}  
class RealImage implements Image {  
    private String filename;  
    public RealImage(String filename) {  
        this.filename = filename;  
        load();  
    }  
    private void load() {  
        System.out.println("Loading " + filename);  
    }  
    public void display() {  
        System.out.println("Displaying " + filename);  
    }  
}  
class ProxyImage implements Image {  
    private RealImage realImage;  
    private String filename;  
    public ProxyImage(String filename) {  
        this.filename = filename;  
    }  
    public void display() {  
        if (realImage == null) {  
            realImage = new RealImage(filename);  
        }  
        realImage.display();  
    }  
}  
// Usage  
Image image = new ProxyImage("photo.jpg");  
image.display();  
```

#### TypeScript  
```typescript  
interface Image {  
    display(): void;  
}  
class RealImage implements Image {  
    constructor(private filename: string) {  
        this.load();  
    }  
    private load() {  
        console.log(`Loading ${this.filename}`);  
    }  
    display() {  
        console.log(`Displaying ${this.filename}`);  
    }  
}  
class ProxyImage implements Image {  
    private realImage: RealImage | null = null;  
    constructor(private filename: string) {}  
    display() {  
        if (!this.realImage) {  
            this.realImage = new RealImage(this.filename);  
        }  
        this.realImage.display();  
    }  
}  
// Usage  
const image = new ProxyImage("photo.jpg");  
image.display();  
```

#### Python  
```python  
class Image:  
    def display(self):  
        raise NotImplementedError  
class RealImage(Image):  
    def __init__(self, filename):  
        self.filename = filename  
        self.load()  
    def load(self):  
        print(f"Loading {self.filename}")  
    def display(self):  
        print(f"Displaying {self.filename}")  
class ProxyImage(Image):  
    def __init__(self, filename):  
        self.filename = filename  
        self.real_image = None  
    def display(self):  
        if self.real_image is None:  
            self.real_image = RealImage(self.filename)  
        self.real_image.display()  
# Usage  
image = ProxyImage("photo.jpg")  
image.display()  
```

#### Go  
```go  
package main  
import "fmt"  
type Image interface {  
    Display()  
}  
type RealImage struct {  
    filename string  
}  
func (r *RealImage) load() {  
    fmt.Printf("Loading %s\n", r.filename)  
}  
func (r *RealImage) Display() {  
    fmt.Printf("Displaying %s\n", r.filename)  
}  
type ProxyImage struct {  
    filename string  
    realImage *RealImage  
}  
func (p *ProxyImage) Display() {  
    if p.realImage == nil {  
        p.realImage = &RealImage{p.filename}  
        p.realImage.load()  
    }  
    p.realImage.Display()  
}  
// Usage  
func main() {  
    image := &ProxyImage{filename: "photo.jpg"}  
    image.Display()  
}  
```

**Key Differences:**  
- Java and TypeScript use interfaces for a clear proxy contract.  
- Python uses duck typing, keeping the proxy simple.  
- Go uses interfaces and composition, focusing on behavior.  
- All control access to the real object effectively.

**WHEN SHOULD YOU USE IT?**  
- When you need to control access to an object (e.g., lazy loading, caching).  
- When you want to add extra logic (e.g., logging, permissions) without modifying the object.  
- When the real object is resource-intensive to create or use.

**WHERE YOU’VE ALREADY SEEN IT**  
`Virtual Proxies` in ORMs like Hibernate (lazy loading of database entities).  
`Security Proxies` in Java RMI (controls access to remote objects).  
`Web Proxies` like NGINX (caching and load balancing for web servers).