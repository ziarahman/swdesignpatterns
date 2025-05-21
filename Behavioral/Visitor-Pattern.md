# Design Patterns: Behavioral - Visitor

## Visitor Pattern - Simple Explanation

**SITUATION: YOU'RE PERFORMING OPERATIONS ON OBJECTS**  
You have a document structure with different elements. 

Initially, you have: 

#### Java
```java
class Document {
    void export() {
        // export logic
    }
}
``` 
All good.

**THEN THE OPERATIONS MULTIPLY**  
Now you have:  
- Different elements (Text, Image, Table)
- Different operations (Export, Print, Validate)
- Different formats (HTML, PDF, Plain)

You're stuck with:
```java
class Text {
    void exportHTML() { }
    void exportPDF() { }
    void print() { }
    void validate() { }
}
// Same for Image, Table...
``` 
Code explosion!

**WHAT'S THE PROBLEM?**  
- Classes bloated with similar operations.
- New operations require changing all classes.
- Related behavior scattered across classes.
- Hard to add new operations.

**HOW VISITOR SAVES YOU**  
Move the operations to separate visitor classes.

You say: "Here's a visitor, it knows how to handle each element type." 

And the Visitor:  
- Separates algorithms from objects.
- Makes adding operations easy.
- Keeps related operations together.
- Allows double dispatch.

Now you just: 
```java
document.accept(new HTMLExportVisitor());
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
// Visitor interface
interface Visitor {
    void visitText(Text text);
    void visitImage(Image image);
    void visitTable(Table table);
}

// Element interface
interface Element {
    void accept(Visitor visitor);
}

// Concrete elements
class Text implements Element {
    private String content;
    
    public Text(String content) {
        this.content = content;
    }
    
    public String getContent() {
        return content;
    }
    
    @Override
    public void accept(Visitor visitor) {
        visitor.visitText(this);
    }
}

class Image implements Element {
    private String path;
    
    public Image(String path) {
        this.path = path;
    }
    
    public String getPath() {
        return path;
    }
    
    @Override
    public void accept(Visitor visitor) {
        visitor.visitImage(this);
    }
}

// Concrete visitors
class HTMLExportVisitor implements Visitor {
    @Override
    public void visitText(Text text) {
        System.out.println("<p>" + text.getContent() + "</p>");
    }
    
    @Override
    public void visitImage(Image image) {
        System.out.println("<img src='" + image.getPath() + "'/>");
    }
    
    @Override
    public void visitTable(Table table) {
        System.out.println("<table>" + table.getData() + "</table>");
    }
}
```

#### TypeScript  
```typescript  
interface Visitor {
    visitText(text: Text): void;
    visitImage(image: Image): void;
    visitTable(table: Table): void;
}

interface Element {
    accept(visitor: Visitor): void;
}

class Text implements Element {
    constructor(private content: string) {}
    
    getContent(): string {
        return this.content;
    }
    
    accept(visitor: Visitor): void {
        visitor.visitText(this);
    }
}

class Image implements Element {
    constructor(private path: string) {}
    
    getPath(): string {
        return this.path;
    }
    
    accept(visitor: Visitor): void {
        visitor.visitImage(this);
    }
}

class HTMLExportVisitor implements Visitor {
    visitText(text: Text): void {
        console.log(`<p>${text.getContent()}</p>`);
    }
    
    visitImage(image: Image): void {
        console.log(`<img src='${image.getPath()}'/>`);
    }
    
    visitTable(table: Table): void {
        console.log(`<table>${table.getData()}</table>`);
    }
}
```

#### Python  
```python  
from abc import ABC, abstractmethod

class Visitor(ABC):
    @abstractmethod
    def visit_text(self, text):
        pass
    
    @abstractmethod
    def visit_image(self, image):
        pass
    
    @abstractmethod
    def visit_table(self, table):
        pass

class Element(ABC):
    @abstractmethod
    def accept(self, visitor: Visitor):
        pass

class Text(Element):
    def __init__(self, content: str):
        self._content = content
    
    @property
    def content(self) -> str:
        return self._content
    
    def accept(self, visitor: Visitor):
        visitor.visit_text(self)

class Image(Element):
    def __init__(self, path: str):
        self._path = path
    
    @property
    def path(self) -> str:
        return self._path
    
    def accept(self, visitor: Visitor):
        visitor.visit_image(self)

class HTMLExportVisitor(Visitor):
    def visit_text(self, text: Text):
        print(f"<p>{text.content}</p>")
    
    def visit_image(self, image: Image):
        print(f"<img src='{image.path}'/>")
    
    def visit_table(self, table):
        print(f"<table>{table.data}</table>")
```

#### Go  
```go  
package main

import "fmt"

// Visitor interface
type Visitor interface {
    VisitText(*Text)
    VisitImage(*Image)
    VisitTable(*Table)
}

// Element interface
type Element interface {
    Accept(Visitor)
}

// Text element
type Text struct {
    content string
}

func (t *Text) Accept(v Visitor) {
    v.VisitText(t)
}

func (t *Text) GetContent() string {
    return t.content
}

// Image element
type Image struct {
    path string
}

func (i *Image) Accept(v Visitor) {
    v.VisitImage(i)
}

func (i *Image) GetPath() string {
    return i.path
}

// HTML Export visitor
type HTMLExportVisitor struct{}

func (v *HTMLExportVisitor) VisitText(t *Text) {
    fmt.Printf("<p>%s</p>\n", t.GetContent())
}

func (v *HTMLExportVisitor) VisitImage(i *Image) {
    fmt.Printf("<img src='%s'/>\n", i.GetPath())
}

func (v *HTMLExportVisitor) VisitTable(t *Table) {
    fmt.Printf("<table>%s</table>\n", t.GetData())
}
```

**Key Differences:**  
- Java uses interface-based visitor pattern.
- TypeScript adds type safety to visitor methods.
- Python uses ABC for abstract visitor definition.
- Go implements visitor through interfaces.

**WHEN SHOULD YOU USE IT?**  
- When you need to perform operations on complex object structures.
- When you want to add new operations without changing objects.
- When operations are not fundamental to the object structure.
- When you have a stable set of element classes.

**WHERE YOU'VE ALREADY SEEN IT**  
- AST processing in compilers
- Document processing (PDF, HTML export)
- GUI framework event handling
- Static code analysis tools
