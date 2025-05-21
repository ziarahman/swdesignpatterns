# Design Patterns: Behavioral - Chain of Responsibility

## Chain of Responsibility Pattern - Simple Explanation

**SITUATION: YOU'RE HANDLING REQUESTS**  
You're building a request handling system like logging or authentication. 

Initially, you have: 

#### Java
```java
if (isDebugLog) {
    logDebug(message);
}
``` 
All good.

**THEN THE HANDLERS GROW**  
Now you have:  
- `DebugLogger`  
- `InfoLogger`  
- `ErrorLogger`  

You're stuck with: 
```java
if (level == DEBUG) { 
    logDebug(message); 
} else if (level == INFO) { 
    logInfo(message); 
} else if (level == ERROR) {
    logError(message);
}
``` 
Everywhere.

**WHAT'S THE PROBLEM?**  
- The handling logic is tightly coupled.  
- Adding a new handler means changing existing code.  
- It violates the Single Responsibility Principle.  
- Hard to change the order of handling.

**HOW CHAIN OF RESPONSIBILITY SAVES YOU**  
Instead of hardcoding, you create a chain of handlers. 

You say: "Here's a request, pass it along the chain until someone handles it." 

And the Chain:  
- Passes request from handler to handler.  
- Each handler decides to process and/or pass along.  
- Keeps handlers decoupled.  
- Easy to add/remove/reorder handlers.  

Now you just: 
```java
Logger chain = new DebugLogger(new InfoLogger(new ErrorLogger(null)));
chain.log(DEBUG, "Debug message");
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
abstract class Logger {
    protected Logger nextLogger;
    protected int level;
    
    public void setNext(Logger next) {
        this.nextLogger = next;
    }
    
    public void log(int level, String message) {
        if (this.level <= level) {
            write(message);
        }
        if (nextLogger != null) {
            nextLogger.log(level, message);
        }
    }
    
    abstract protected void write(String message);
}

class DebugLogger extends Logger {
    public DebugLogger() {
        this.level = 1;
    }
    
    protected void write(String message) {
        System.out.println("DEBUG: " + message);
    }
}

class InfoLogger extends Logger {
    public InfoLogger() {
        this.level = 2;
    }
    
    protected void write(String message) {
        System.out.println("INFO: " + message);
    }
}
```

#### TypeScript  
```typescript  
abstract class Logger {
    protected next: Logger | null = null;
    protected level: number;

    setNext(next: Logger): Logger {
        this.next = next;
        return next;
    }

    log(level: number, message: string): void {
        if (this.level <= level) {
            this.write(message);
        }
        if (this.next) {
            this.next.log(level, message);
        }
    }

    abstract write(message: string): void;
}

class DebugLogger extends Logger {
    constructor() {
        super();
        this.level = 1;
    }

    write(message: string): void {
        console.log(`DEBUG: ${message}`);
    }
}

class InfoLogger extends Logger {
    constructor() {
        super();
        this.level = 2;
    }

    write(message: string): void {
        console.log(`INFO: ${message}`);
    }
}
```

#### Python  
```python  
from abc import ABC, abstractmethod

class Logger(ABC):
    def __init__(self):
        self._next = None
        self.level = None
    
    def set_next(self, next_logger):
        self._next = next_logger
        return next_logger
    
    def log(self, level, message):
        if self.level <= level:
            self.write(message)
        if self._next:
            self._next.log(level, message)
    
    @abstractmethod
    def write(self, message):
        pass

class DebugLogger(Logger):
    def __init__(self):
        super().__init__()
        self.level = 1
    
    def write(self, message):
        print(f"DEBUG: {message}")

class InfoLogger(Logger):
    def __init__(self):
        super().__init__()
        self.level = 2
    
    def write(self, message):
        print(f"INFO: {message}")
```

#### Go  
```go  
package main

import "fmt"

type Logger interface {
    SetNext(logger Logger)
    Log(level int, message string)
}

type BaseLogger struct {
    next  Logger
    level int
}

func (l *BaseLogger) SetNext(next Logger) {
    l.next = next
}

func (l *BaseLogger) Log(level int, message string) {
    if l.level <= level {
        l.Write(message)
    }
    if l.next != nil {
        l.next.Log(level, message)
    }
}

type DebugLogger struct {
    BaseLogger
}

func NewDebugLogger() *DebugLogger {
    return &DebugLogger{BaseLogger{level: 1}}
}

func (l *DebugLogger) Write(message string) {
    fmt.Printf("DEBUG: %s\n", message)
}

type InfoLogger struct {
    BaseLogger
}

func NewInfoLogger() *InfoLogger {
    return &InfoLogger{BaseLogger{level: 2}}
}

func (l *InfoLogger) Write(message string) {
    fmt.Printf("INFO: %s\n", message)
}
```

**Key Differences:**  
- Java and TypeScript use abstract classes for the base handler.
- Python uses ABC (Abstract Base Class) for the interface.
- Go uses interfaces and struct embedding.
- All maintain the chain through references to the next handler.

**WHEN SHOULD YOU USE IT?**  
- When you need multiple objects to handle a request in sequence.
- When the handler isn't known beforehand.
- When you want to decouple senders and receivers.
- When the set of handlers can change dynamically.

**WHERE YOU'VE ALREADY SEEN IT**  
- Exception handling in try-catch blocks
- Event bubbling in DOM
- Middleware in web frameworks
- Logging frameworks
