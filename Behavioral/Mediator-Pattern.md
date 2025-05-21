# Design Patterns: Behavioral - Mediator

## Mediator Pattern - Simple Explanation

**SITUATION: YOU'RE MANAGING COMPONENT INTERACTIONS**  
You're building a UI with multiple components that interact. 

Initially, you have: 

#### Java
```java
button.addListener(() -> textField.clear());
``` 
All good.

**THEN THE INTERACTIONS MULTIPLY**  
Now you have:  
- Buttons
- Text Fields
- Checkboxes
- Lists

All talking to each other:
```java
button1.addListener(() -> {
    textField.clear();
    checkbox.setChecked(false);
    list.selectFirst();
});
``` 
Chaos ensues.

**WHAT'S THE PROBLEM?**  
- Components are tightly coupled to each other.
- Each component needs to know about others.
- Changes in one affect many.
- Testing becomes complicated.

**HOW MEDIATOR SAVES YOU**  
Instead of direct communication, components talk through a mediator.

You say: "Talk to the mediator, it'll handle the rest." 

And the Mediator:  
- Centralizes communication logic.
- Keeps components decoupled.
- Makes changes easier.
- Simplifies testing.

Now you just: 
```java
// Components only know about the mediator
button.click(); // Mediator handles what happens next
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
// Mediator interface
interface UIMediator {
    void notify(String event, UIComponent sender);
}

// Abstract component
abstract class UIComponent {
    protected UIMediator mediator;
    
    public void setMediator(UIMediator mediator) {
        this.mediator = mediator;
    }
}

// Concrete components
class Button extends UIComponent {
    public void click() {
        mediator.notify("click", this);
    }
}

class TextField extends UIComponent {
    private String text = "";
    
    public void clear() {
        text = "";
        mediator.notify("clear", this);
    }
    
    public void setText(String text) {
        this.text = text;
        mediator.notify("textChanged", this);
    }
}

// Concrete mediator
class DialogMediator implements UIMediator {
    private Button button;
    private TextField textField;
    
    public void setButton(Button button) {
        this.button = button;
        button.setMediator(this);
    }
    
    public void setTextField(TextField textField) {
        this.textField = textField;
        textField.setMediator(this);
    }
    
    public void notify(String event, UIComponent sender) {
        if (event.equals("click")) {
            textField.clear();
        } else if (event.equals("textChanged")) {
            button.setEnabled(!textField.isEmpty());
        }
    }
}
```

#### TypeScript  
```typescript  
interface Mediator {
    notify(event: string, sender: Component): void;
}

abstract class Component {
    protected mediator: Mediator;

    constructor(mediator: Mediator) {
        this.mediator = mediator;
    }
}

class Button extends Component {
    click(): void {
        this.mediator.notify('click', this);
    }
}

class TextField extends Component {
    private text: string = '';

    clear(): void {
        this.text = '';
        this.mediator.notify('clear', this);
    }

    setText(text: string): void {
        this.text = text;
        this.mediator.notify('textChanged', this);
    }
}

class DialogMediator implements Mediator {
    private button: Button;
    private textField: TextField;

    constructor() {
        this.button = new Button(this);
        this.textField = new TextField(this);
    }

    notify(event: string, sender: Component): void {
        if (event === 'click') {
            this.textField.clear();
        } else if (event === 'textChanged') {
            // Update button state based on text
        }
    }
}
```

#### Python  
```python  
from abc import ABC, abstractmethod

class Mediator(ABC):
    @abstractmethod
    def notify(self, event: str, sender: object) -> None:
        pass

class Component:
    def __init__(self, mediator: Mediator = None):
        self._mediator = mediator

    @property
    def mediator(self) -> Mediator:
        return self._mediator

    @mediator.setter
    def mediator(self, mediator: Mediator) -> None:
        self._mediator = mediator

class Button(Component):
    def click(self):
        self.mediator.notify("click", self)

class TextField(Component):
    def __init__(self, mediator: Mediator = None):
        super().__init__(mediator)
        self._text = ""

    def clear(self):
        self._text = ""
        self.mediator.notify("clear", self)

    def set_text(self, text: str):
        self._text = text
        self.mediator.notify("textChanged", self)

class DialogMediator(Mediator):
    def __init__(self):
        self._button = Button()
        self._text_field = TextField()
        self._button.mediator = self
        self._text_field.mediator = self

    def notify(self, event: str, sender: object):
        if event == "click":
            self._text_field.clear()
        elif event == "textChanged":
            # Update button state
            pass
```

#### Go  
```go  
package main

import "fmt"

// Mediator interface
type Mediator interface {
    Notify(event string, sender interface{})
}

// Component interface
type Component interface {
    SetMediator(mediator Mediator)
}

// Button component
type Button struct {
    mediator Mediator
}

func (b *Button) SetMediator(mediator Mediator) {
    b.mediator = mediator
}

func (b *Button) Click() {
    b.mediator.Notify("click", b)
}

// TextField component
type TextField struct {
    mediator Mediator
    text     string
}

func (t *TextField) SetMediator(mediator Mediator) {
    t.mediator = mediator
}

func (t *TextField) Clear() {
    t.text = ""
    t.mediator.Notify("clear", t)
}

func (t *TextField) SetText(text string) {
    t.text = text
    t.mediator.Notify("textChanged", t)
}

// Concrete mediator
type DialogMediator struct {
    button    *Button
    textField *TextField
}

func NewDialogMediator() *DialogMediator {
    mediator := &DialogMediator{
        button:    &Button{},
        textField: &TextField{},
    }
    mediator.button.SetMediator(mediator)
    mediator.textField.SetMediator(mediator)
    return mediator
}

func (d *DialogMediator) Notify(event string, sender interface{}) {
    switch event {
    case "click":
        d.textField.Clear()
    case "textChanged":
        // Update button state
    }
}
```

**Key Differences:**  
- Java uses interfaces and abstract classes for flexibility.
- TypeScript leverages type system for component relationships.
- Python uses ABC for abstract mediator definition.
- Go uses interfaces and composition for mediation.

**WHEN SHOULD YOU USE IT?**  
- When components need to communicate but shouldn't know about each other.
- When you want to reduce coupling between components.
- When component relationships become too complex.
- When you want to reuse components in different contexts.

**WHERE YOU'VE ALREADY SEEN IT**  
- Air traffic control systems
- Chat room servers
- GUI frameworks
- Event management systems
