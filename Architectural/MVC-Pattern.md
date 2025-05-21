# Design Patterns: Architectural - MVC (Model-View-Controller)

## MVC Pattern - Simple Explanation

**SITUATION: YOU'RE BUILDING A USER INTERFACE**  
You have a simple app displaying user data. 

Initially, you have: 

#### Java
```java
class UserScreen {
    private User user;
    
    void updateUser(String name) {
        user.setName(name);
        repaintScreen();
    }
}
``` 
All good.

**THEN THE COMPLEXITY GROWS**  
Now you have:  
- Multiple views of the same data
- Complex business logic
- Different user interactions
- Data validation rules

You're stuck with: 
```java
class UserScreen {
    private User user;
    private Database db;
    
    void updateUser(String name) {
        if (validateName(name)) {  // Business logic
            user.setName(name);     // Data update
            db.save(user);          // Persistence
            repaintScreen();        // UI update
            updateOtherScreens();   // Other views
        }
    }
}
``` 
Everything mixed together!

**WHAT'S THE PROBLEM?**  
- Business logic mixed with UI code
- Hard to maintain and test
- Changes affect multiple parts
- Can't reuse components

**HOW MVC SAVES YOU**  
Split your app into three parts:
- Model: Data and business rules
- View: Display and user interface
- Controller: Handles user input

You say: "Each part has its own responsibility." 

And MVC:  
- Separates concerns
- Makes code maintainable
- Enables parallel development
- Allows component reuse

Now you just: 
```java
// Model handles data and business rules
user.setName(name);

// Controller handles user input
controller.updateUser(name);

// View displays the data
view.render(user);
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
// Model
class UserModel {
    private String name;
    private String email;
    
    public void setName(String name) {
        this.name = name;
        notifyObservers();
    }
    
    public String getName() {
        return name;
    }
    
    private void notifyObservers() {
        // Notify views of change
    }
}

// View
class UserView {
    private UserController controller;
    
    public void setController(UserController controller) {
        this.controller = controller;
    }
    
    public void render(UserModel user) {
        System.out.println("Name: " + user.getName());
    }
    
    public void handleUserInput(String name) {
        controller.updateUser(name);
    }
}

// Controller
class UserController {
    private UserModel model;
    private UserView view;
    
    public UserController(UserModel model, UserView view) {
        this.model = model;
        this.view = view;
        this.view.setController(this);
    }
    
    public void updateUser(String name) {
        if (validateName(name)) {
            model.setName(name);
            view.render(model);
        }
    }
    
    private boolean validateName(String name) {
        return name != null && !name.trim().isEmpty();
    }
}
```

#### TypeScript  
```typescript  
// Model
class UserModel {
    private name: string;
    private email: string;
    private observers: UserView[] = [];
    
    addObserver(observer: UserView): void {
        this.observers.push(observer);
    }
    
    setName(name: string): void {
        this.name = name;
        this.notifyObservers();
    }
    
    getName(): string {
        return this.name;
    }
    
    private notifyObservers(): void {
        this.observers.forEach(observer => observer.render(this));
    }
}

// View
class UserView {
    private controller: UserController;
    
    setController(controller: UserController): void {
        this.controller = controller;
    }
    
    render(user: UserModel): void {
        console.log(`Name: ${user.getName()}`);
    }
    
    handleInput(name: string): void {
        this.controller.updateUser(name);
    }
}

// Controller
class UserController {
    constructor(
        private model: UserModel,
        private view: UserView
    ) {
        this.view.setController(this);
        this.model.addObserver(this.view);
    }
    
    updateUser(name: string): void {
        if (this.validateName(name)) {
            this.model.setName(name);
        }
    }
    
    private validateName(name: string): boolean {
        return name != null && name.trim().length > 0;
    }
}
```

#### Python  
```python  
from abc import ABC, abstractmethod
from typing import List

# Model
class UserModel:
    def __init__(self):
        self._name: str = ""
        self._observers: List['UserView'] = []
    
    def add_observer(self, observer: 'UserView') -> None:
        self._observers.append(observer)
    
    @property
    def name(self) -> str:
        return self._name
    
    @name.setter
    def name(self, value: str) -> None:
        self._name = value
        self._notify_observers()
    
    def _notify_observers(self) -> None:
        for observer in self._observers:
            observer.render(self)

# View
class UserView:
    def __init__(self):
        self.controller: 'UserController' = None
    
    def set_controller(self, controller: 'UserController') -> None:
        self.controller = controller
    
    def render(self, user: UserModel) -> None:
        print(f"Name: {user.name}")
    
    def handle_input(self, name: str) -> None:
        if self.controller:
            self.controller.update_user(name)

# Controller
class UserController:
    def __init__(self, model: UserModel, view: UserView):
        self.model = model
        self.view = view
        self.view.set_controller(self)
        self.model.add_observer(self.view)
    
    def update_user(self, name: str) -> None:
        if self._validate_name(name):
            self.model.name = name
    
    def _validate_name(self, name: str) -> bool:
        return name is not None and len(name.strip()) > 0
```

#### Go  
```go  
package main

import "fmt"

// Model
type UserModel struct {
    name      string
    observers []UserView
}

func (m *UserModel) AddObserver(view UserView) {
    m.observers = append(m.observers, view)
}

func (m *UserModel) SetName(name string) {
    m.name = name
    m.notifyObservers()
}

func (m *UserModel) GetName() string {
    return m.name
}

func (m *UserModel) notifyObservers() {
    for _, observer := range m.observers {
        observer.Render(m)
    }
}

// View
type UserView struct {
    controller *UserController
}

func (v *UserView) SetController(controller *UserController) {
    v.controller = controller
}

func (v *UserView) Render(model *UserModel) {
    fmt.Printf("Name: %s\n", model.GetName())
}

func (v *UserView) HandleInput(name string) {
    v.controller.UpdateUser(name)
}

// Controller
type UserController struct {
    model *UserModel
    view  *UserView
}

func NewUserController(model *UserModel, view *UserView) *UserController {
    controller := &UserController{
        model: model,
        view:  view,
    }
    view.SetController(controller)
    model.AddObserver(*view)
    return controller
}

func (c *UserController) UpdateUser(name string) {
    if c.validateName(name) {
        c.model.SetName(name)
    }
}

func (c *UserController) validateName(name string) bool {
    return len(name) > 0
}
```

**Key Differences:**  
- Java uses traditional OOP with explicit observer pattern
- TypeScript leverages type system for relationships
- Python uses properties and type hints
- Go implements through composition and interfaces

**WHEN SHOULD YOU USE IT?**  
- When building user interfaces
- When you need multiple views of the same data
- When business logic should be separate from UI
- When you want to enable parallel development

**WHERE YOU'VE ALREADY SEEN IT**  
- Web frameworks (Django, Rails)
- Desktop applications (Java Swing)
- Mobile development (iOS, Android)
- Enterprise applications
