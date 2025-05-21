# Design Patterns: Behavioral - Command

## Command Pattern - Simple Explanation

**SITUATION: YOU’RE HANDLING USER ACTIONS**  
You’re building a text editor with actions like `Copy`, `Paste`. Initially, you call: 

#### Java
```java
editor.copy(); 
editor.paste();
``` 
All good.

**THEN THE FEATURES GROW**  
Now you need:  
- Undo/redo support.  
- Logging actions.  
- Queuing actions.  

You’re stuck adding logic to each method: 

```java
editor.copy(); 
log("copy"); 
undoStack.push("copy");
``` 
Everywhere.

**WHAT’S THE PROBLEM?**  
- Actions are tightly coupled to their execution.  
- Adding undo, logging, or queuing means changing every action.  
- It’s hard to manage action history or schedule tasks.  
- The code becomes cluttered and hard to maintain.

**HOW COMMAND SAVES YOU**  
Instead of direct calls, you create `Command` objects like `CopyCommand`, `PasteCommand`. 

You say: “Hey Command, execute this action.” 

And the Command:  
- Encapsulates the action as an object.  
- Supports undo/redo, logging, or queuing easily.  
- Decouples the action from its execution.  

Now you just: 
```java
Command copy = new CopyCommand(editor); 
copy.execute();
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
interface Command {  
    void execute();  
    void undo();  
}  
class Editor {  
    public void copy() {  
        System.out.println("Copying");  
    }  
    public void paste() {  
        System.out.println("Pasting");  
    }  
}  
class CopyCommand implements Command {  
    private Editor editor;  
    public CopyCommand(Editor editor) {  
        this.editor = editor;  
    }  
    public void execute() {  
        editor.copy();  
    }  
    public void undo() {  
        System.out.println("Undoing copy");  
    }  
}  
class PasteCommand implements Command {  
    private Editor editor;  
    public PasteCommand(Editor editor) {  
        this.editor = editor;  
    }  
    public void execute() {  
        editor.paste();  
    }  
    public void undo() {  
        System.out.println("Undoing paste");  
    }  
}  
// Usage  
Editor editor = new Editor();  
Command copy = new CopyCommand(editor);  
copy.execute();  
```

#### TypeScript  
```typescript  
interface Command {  
    execute(): void;  
    undo(): void;  
}  
class Editor {  
    copy() {  
        console.log("Copying");  
    }  
    paste() {  
        console.log("Pasting");  
    }  
}  
class CopyCommand implements Command {  
    constructor(private editor: Editor) {}  
    execute() {  
        this.editor.copy();  
    }  
    undo() {  
        console.log("Undoing copy");  
    }  
}  
class PasteCommand implements Command {  
    constructor(private editor: Editor) {}  
    execute() {  
        this.editor.paste();  
    }  
    undo() {  
        console.log("Undoing paste");  
    }  
}  
// Usage  
const editor = new Editor();  
const copy = new CopyCommand(editor);  
copy.execute();  
```

#### Python  
```python  
from abc import ABC, abstractmethod  
class Command(ABC):  
    @abstractmethod  
    def execute(self):  
        pass  
    @abstractmethod  
    def undo(self):  
        pass  
class Editor:  
    def copy(self):  
        print("Copying")  
    def paste(self):  
        print("Pasting")  
class CopyCommand(Command):  
    def __init__(self, editor):  
        self.editor = editor  
    def execute(self):  
        self.editor.copy()  
    def undo(self):  
        print("Undoing copy")  
class PasteCommand(Command):  
    def __init__(self, editor):  
        self.editor = editor  
    def execute(self):  
        self.editor.paste()  
    def undo(self):  
        print("Undoing paste")  
# Usage  
editor = Editor()  
copy = CopyCommand(editor)  
copy.execute()  
```

#### Go  
```go  
package main  
import "fmt"  
type Command interface {  
    Execute()  
    Undo()  
}  
type Editor struct{}  
func (e *Editor) Copy() {  
    fmt.Println("Copying")  
}  
func (e *Editor) Paste() {  
    fmt.Println("Pasting")  
}  
type CopyCommand struct{ editor *Editor }  
func (c *CopyCommand) Execute() {  
    c.editor.Copy()  
}  
func (c *CopyCommand) Undo() {  
    fmt.Println("Undoing copy")  
}  
type PasteCommand struct{ editor *Editor }  
func (c *PasteCommand) Execute() {  
    c.editor.Paste()  
}  
func (c *PasteCommand) Undo() {  
    fmt.Println("Undoing paste")  
}  
// Usage  
func main() {  
    editor := &Editor{}  
    copy := &CopyCommand{editor}  
    copy.Execute()  
}  
```

**Key Differences:**  
- Java and TypeScript use interfaces for a structured command hierarchy.  
- Python uses abstract base classes or duck typing for flexibility.  
- Go uses interfaces and composition, keeping it simple.  
- All encapsulate actions for extensibility.

**WHEN SHOULD YOU USE IT?**  
- When you need to encapsulate actions as objects.  
- When you want to support undo/redo or logging.  
- When you need to queue or schedule actions.

**WHERE YOU’VE ALREADY SEEN IT**  
`Runnable` in Java (for thread execution).  
`Action` in GUI frameworks (for button clicks).