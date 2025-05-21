# Design Patterns: Behavioral - Template Method

## Template Method Pattern - Simple Explanation

**SITUATION: YOU'RE BUILDING SIMILAR PROCESSES**  
You're creating different types of documents (reports, resumes). 

Initially, you have: 

#### Java
```java
class Report {
    void generate() {
        // specific report logic
    }
}
``` 
All good.

**THEN THE PROCESSES MULTIPLY**  
Now you have:  
- `PDFReport`  
- `ExcelReport`  
- `WordReport`  

You notice they all follow similar steps:
1. Open file
2. Add content
3. Format
4. Save

You're stuck duplicating code.

**WHAT'S THE PROBLEM?**  
- Common steps are duplicated across classes.
- Changes to the process require updating multiple classes.
- Hard to ensure all implementations follow the same steps.
- Violates DRY (Don't Repeat Yourself).

**HOW TEMPLATE METHOD SAVES YOU**  
Define the skeleton of the algorithm in a base class, but let subclasses override specific steps.

You say: "Here's the process template, customize only what you need." 

And the Template:  
- Defines the sequence of steps.
- Lets subclasses override only necessary steps.
- Keeps the process structure consistent.
- Eliminates code duplication.

Now you just: 
```java
Report report = new PDFReport();
report.generate(); // follows template, with PDF-specific steps
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
abstract class Report {
    // Template method
    public final void generate() {
        openFile();
        addContent();
        format();
        save();
    }
    
    // These can be overridden
    protected abstract void openFile();
    protected abstract void addContent();
    
    // This is a hook - optional override
    protected void format() {
        System.out.println("Default formatting");
    }
    
    // This is fixed for all reports
    private void save() {
        System.out.println("Saving report...");
    }
}

class PDFReport extends Report {
    protected void openFile() {
        System.out.println("Opening PDF file");
    }
    
    protected void addContent() {
        System.out.println("Adding PDF content");
    }
    
    protected void format() {
        System.out.println("PDF formatting");
    }
}

class ExcelReport extends Report {
    protected void openFile() {
        System.out.println("Opening Excel file");
    }
    
    protected void addContent() {
        System.out.println("Adding Excel content");
    }
}
```

#### TypeScript  
```typescript  
abstract class Report {
    // Template method
    public generate(): void {
        this.openFile();
        this.addContent();
        this.format();
        this.save();
    }
    
    protected abstract openFile(): void;
    protected abstract addContent(): void;
    
    // Hook method
    protected format(): void {
        console.log("Default formatting");
    }
    
    private save(): void {
        console.log("Saving report...");
    }
}

class PDFReport extends Report {
    protected openFile(): void {
        console.log("Opening PDF file");
    }
    
    protected addContent(): void {
        console.log("Adding PDF content");
    }
    
    protected format(): void {
        console.log("PDF formatting");
    }
}

class ExcelReport extends Report {
    protected openFile(): void {
        console.log("Opening Excel file");
    }
    
    protected addContent(): void {
        console.log("Adding Excel content");
    }
}
```

#### Python  
```python  
from abc import ABC, abstractmethod

class Report(ABC):
    # Template method
    def generate(self):
        self.open_file()
        self.add_content()
        self.format()
        self._save()
    
    @abstractmethod
    def open_file(self):
        pass
    
    @abstractmethod
    def add_content(self):
        pass
    
    # Hook method
    def format(self):
        print("Default formatting")
    
    # Fixed for all reports
    def _save(self):
        print("Saving report...")

class PDFReport(Report):
    def open_file(self):
        print("Opening PDF file")
    
    def add_content(self):
        print("Adding PDF content")
    
    def format(self):
        print("PDF formatting")

class ExcelReport(Report):
    def open_file(self):
        print("Opening Excel file")
    
    def add_content(self):
        print("Adding Excel content")
```

#### Go  
```go  
package main

import "fmt"

// Since Go doesn't have inheritance, we use composition
type Report interface {
    OpenFile()
    AddContent()
    Format()
    Generate()
}

// Base template implementation
type BaseReport struct {
    report Report
}

func (b *BaseReport) Generate() {
    b.report.OpenFile()
    b.report.AddContent()
    b.report.Format()
    b.save()
}

func (b *BaseReport) Format() {
    fmt.Println("Default formatting")
}

func (b *BaseReport) save() {
    fmt.Println("Saving report...")
}

// PDF Report implementation
type PDFReport struct {
    BaseReport
}

func NewPDFReport() *PDFReport {
    pdf := &PDFReport{}
    pdf.BaseReport.report = pdf
    return pdf
}

func (p *PDFReport) OpenFile() {
    fmt.Println("Opening PDF file")
}

func (p *PDFReport) AddContent() {
    fmt.Println("Adding PDF content")
}

func (p *PDFReport) Format() {
    fmt.Println("PDF formatting")
}
```

**Key Differences:**  
- Java and TypeScript use abstract classes and inheritance.
- Python uses ABC for abstract methods.
- Go uses interfaces and composition since it doesn't support inheritance.
- All maintain the template structure while allowing customization.

**WHEN SHOULD YOU USE IT?**  
- When you have several algorithms that are similar in structure.
- When you want to avoid code duplication in similar algorithms.
- When you want to control how subclasses extend your algorithm.
- When you have invariant parts of an algorithm that should stay fixed.

**WHERE YOU'VE ALREADY SEEN IT**  
- Framework callback methods (onCreate, onDestroy in Android)
- Java's AbstractList, AbstractSet
- Unit testing setUp/tearDown methods
- Web framework request handling
