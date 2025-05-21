# Design Patterns: Behavioral - Memento

## Memento Pattern - Simple Explanation

**SITUATION: YOU NEED TO SAVE STATE**  
You're building a text editor with undo functionality. 

Initially, you have: 

#### Java
```java
class Editor {
    private String content;
    public void write(String text) {
        content = text;
    }
}
``` 
All good.

**THEN THE REQUIREMENTS GROW**  
Now you need:  
- Undo/Redo
- Save snapshots
- Restore previous states
- Multiple backup points

You're stuck with:
```java
class Editor {
    private List<String> history;
    private String content;
    
    public void save() {
        history.add(content);
    }
    // Getting messy...
}
``` 

**WHAT'S THE PROBLEM?**  
- State management is mixed with business logic.
- Saving state requires exposing internals.
- Hard to add new state properties.
- No encapsulation of state.

**HOW MEMENTO SAVES YOU**  
Create a snapshot object that holds the state.

You say: "Save your state in this memento, I'll keep it safe." 

And the Memento:  
- Captures object's internal state.
- Protects state encapsulation.
- Makes state restoration easy.
- Separates concerns.

Now you just: 
```java
Memento snapshot = editor.save();
// ... later ...
editor.restore(snapshot);
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
// Memento - holds the state
class EditorMemento {
    private final String content;
    private final int cursorPosition;
    
    public EditorMemento(String content, int cursorPosition) {
        this.content = content;
        this.cursorPosition = cursorPosition;
    }
    
    // Only the originator can access state
    private String getContent() {
        return content;
    }
    
    private int getCursorPosition() {
        return cursorPosition;
    }
}

// Originator - the object whose state we want to save
class Editor {
    private String content;
    private int cursorPosition;
    
    public void write(String text) {
        content = text;
        cursorPosition = text.length();
    }
    
    public EditorMemento save() {
        return new EditorMemento(content, cursorPosition);
    }
    
    public void restore(EditorMemento memento) {
        content = memento.getContent();
        cursorPosition = memento.getCursorPosition();
    }
    
    public String getContent() {
        return content;
    }
}

// Caretaker - manages saved states
class History {
    private List<EditorMemento> states = new ArrayList<>();
    
    public void push(EditorMemento state) {
        states.add(state);
    }
    
    public EditorMemento pop() {
        if (states.isEmpty()) return null;
        int lastIndex = states.size() - 1;
        EditorMemento lastState = states.get(lastIndex);
        states.remove(lastIndex);
        return lastState;
    }
}
```

#### TypeScript  
```typescript  
class EditorMemento {
    private content: string;
    private cursorPosition: number;

    constructor(content: string, cursorPosition: number) {
        this.content = content;
        this.cursorPosition = cursorPosition;
    }

    getState(): { content: string; cursorPosition: number } {
        return {
            content: this.content,
            cursorPosition: this.cursorPosition
        };
    }
}

class Editor {
    private content: string = '';
    private cursorPosition: number = 0;

    write(text: string): void {
        this.content = text;
        this.cursorPosition = text.length;
    }

    save(): EditorMemento {
        return new EditorMemento(this.content, this.cursorPosition);
    }

    restore(memento: EditorMemento): void {
        const state = memento.getState();
        this.content = state.content;
        this.cursorPosition = state.cursorPosition;
    }

    getContent(): string {
        return this.content;
    }
}

class History {
    private states: EditorMemento[] = [];

    push(state: EditorMemento): void {
        this.states.push(state);
    }

    pop(): EditorMemento | undefined {
        return this.states.pop();
    }
}
```

#### Python  
```python  
from dataclasses import dataclass
from typing import List

@dataclass
class EditorMemento:
    _content: str
    _cursor_position: int

    @property
    def state(self):
        return {
            'content': self._content,
            'cursor_position': self._cursor_position
        }

class Editor:
    def __init__(self):
        self._content = ""
        self._cursor_position = 0
    
    def write(self, text: str):
        self._content = text
        self._cursor_position = len(text)
    
    def save(self) -> EditorMemento:
        return EditorMemento(self._content, self._cursor_position)
    
    def restore(self, memento: EditorMemento):
        state = memento.state
        self._content = state['content']
        self._cursor_position = state['cursor_position']
    
    @property
    def content(self) -> str:
        return self._content

class History:
    def __init__(self):
        self._states: List[EditorMemento] = []
    
    def push(self, state: EditorMemento):
        self._states.append(state)
    
    def pop(self) -> EditorMemento:
        if self._states:
            return self._states.pop()
        return None
```

#### Go  
```go  
package main

import "fmt"

// Memento holds editor state
type EditorMemento struct {
    content        string
    cursorPosition int
}

// Editor is the originator
type Editor struct {
    content        string
    cursorPosition int
}

func (e *Editor) Write(text string) {
    e.content = text
    e.cursorPosition = len(text)
}

func (e *Editor) Save() *EditorMemento {
    return &EditorMemento{
        content:        e.content,
        cursorPosition: e.cursorPosition,
    }
}

func (e *Editor) Restore(m *EditorMemento) {
    e.content = m.content
    e.cursorPosition = m.cursorPosition
}

func (e *Editor) GetContent() string {
    return e.content
}

// History is the caretaker
type History struct {
    states []*EditorMemento
}

func (h *History) Push(state *EditorMemento) {
    h.states = append(h.states, state)
}

func (h *History) Pop() *EditorMemento {
    if len(h.states) == 0 {
        return nil
    }
    lastIndex := len(h.states) - 1
    state := h.states[lastIndex]
    h.states = h.states[:lastIndex]
    return state
}
```

**Key Differences:**  
- Java uses private access to protect memento state.
- TypeScript uses a getter method for state access.
- Python uses dataclass and properties for clean syntax.
- Go keeps it simple with struct-based implementation.

**WHEN SHOULD YOU USE IT?**  
- When you need to create snapshots of object state.
- When direct access to object fields violates encapsulation.
- When you need undo/redo functionality.
- When you want to implement checkpoints/save points.

**WHERE YOU'VE ALREADY SEEN IT**  
- Text editors' undo mechanism
- Database transactions
- Game save states
- Version control systems
