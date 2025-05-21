# Design Patterns: Behavioral - Null Object

## Null Object Pattern - Simple Explanation

**SITUATION: YOU'RE HANDLING NULL CHECKS**  
You have a logger in your application. 

Initially, you have: 

#### Java
```java
if (logger != null) {
    logger.log("Some message");
}
``` 
All good.

**THEN THE NULL CHECKS MULTIPLY**  
Now you have:  
- Null checks everywhere
- Default behaviors
- Fallback logic

You're stuck with: 
```java
if (logger != null) {
    logger.log(message);
} else {
    // Do nothing or default behavior
}
``` 
Everywhere!

**WHAT'S THE PROBLEM?**  
- Code cluttered with null checks
- Repetitive default behavior
- Risk of NullPointerException
- Hard to maintain consistent fallback logic

**HOW NULL OBJECT SAVES YOU**  
Create a "do nothing" version of your object.

You say: "Use this when you have no real object." 

And the Null Object:  
- Implements the same interface
- Provides "do nothing" behavior
- Never throws NullPointerException
- Makes client code cleaner

Now you just: 
```java
Logger logger = getLogger() ?? new NullLogger();
logger.log(message); // Always safe
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
interface Logger {
    void log(String message);
    void error(String message);
    boolean isNull();
}

class ConsoleLogger implements Logger {
    @Override
    public void log(String message) {
        System.out.println("Log: " + message);
    }
    
    @Override
    public void error(String message) {
        System.err.println("Error: " + message);
    }
    
    @Override
    public boolean isNull() {
        return false;
    }
}

class NullLogger implements Logger {
    @Override
    public void log(String message) {
        // Do nothing
    }
    
    @Override
    public void error(String message) {
        // Do nothing
    }
    
    @Override
    public boolean isNull() {
        return true;
    }
}

class LoggerFactory {
    public static Logger getLogger(String type) {
        if ("console".equals(type)) {
            return new ConsoleLogger();
        }
        return new NullLogger(); // Instead of returning null
    }
}
```

#### TypeScript  
```typescript  
interface Logger {
    log(message: string): void;
    error(message: string): void;
    isNull(): boolean;
}

class ConsoleLogger implements Logger {
    log(message: string): void {
        console.log(`Log: ${message}`);
    }
    
    error(message: string): void {
        console.error(`Error: ${message}`);
    }
    
    isNull(): boolean {
        return false;
    }
}

class NullLogger implements Logger {
    log(message: string): void {
        // Do nothing
    }
    
    error(message: string): void {
        // Do nothing
    }
    
    isNull(): boolean {
        return true;
    }
}

class LoggerFactory {
    static getLogger(type?: string): Logger {
        if (type === 'console') {
            return new ConsoleLogger();
        }
        return new NullLogger();
    }
}
```

#### Python  
```python  
from abc import ABC, abstractmethod

class Logger(ABC):
    @abstractmethod
    def log(self, message: str) -> None:
        pass
    
    @abstractmethod
    def error(self, message: str) -> None:
        pass
    
    @abstractmethod
    def is_null(self) -> bool:
        pass

class ConsoleLogger(Logger):
    def log(self, message: str) -> None:
        print(f"Log: {message}")
    
    def error(self, message: str) -> None:
        print(f"Error: {message}")
    
    def is_null(self) -> bool:
        return False

class NullLogger(Logger):
    def log(self, message: str) -> None:
        pass  # Do nothing
    
    def error(self, message: str) -> None:
        pass  # Do nothing
    
    def is_null(self) -> bool:
        return True

class LoggerFactory:
    @staticmethod
    def get_logger(logger_type: str = None) -> Logger:
        if logger_type == "console":
            return ConsoleLogger()
        return NullLogger()
```

#### Go  
```go  
package main

import "fmt"

// Logger interface
type Logger interface {
    Log(message string)
    Error(message string)
    IsNull() bool
}

// ConsoleLogger implementation
type ConsoleLogger struct{}

func (l *ConsoleLogger) Log(message string) {
    fmt.Printf("Log: %s\n", message)
}

func (l *ConsoleLogger) Error(message string) {
    fmt.Printf("Error: %s\n", message)
}

func (l *ConsoleLogger) IsNull() bool {
    return false
}

// NullLogger implementation
type NullLogger struct{}

func (l *NullLogger) Log(message string) {
    // Do nothing
}

func (l *NullLogger) Error(message string) {
    // Do nothing
}

func (l *NullLogger) IsNull() bool {
    return true
}

// LoggerFactory for creating loggers
func GetLogger(loggerType string) Logger {
    if loggerType == "console" {
        return &ConsoleLogger{}
    }
    return &NullLogger{}
}
```

**Key Differences:**  
- Java uses interface with isNull() method
- TypeScript leverages type system for null safety
- Python uses ABC for abstract logger definition
- Go implements through interfaces with zero-value pattern

**WHEN SHOULD YOU USE IT?**  
- When you need to handle null cases gracefully
- When you want to avoid null checks in client code
- When you need a default "do nothing" behavior
- When null values are expected in your domain

**WHERE YOU'VE ALREADY SEEN IT**  
- Logging frameworks
- UI frameworks (NullLayout)
- Collection libraries (EmptyList, EmptySet)
- Testing frameworks (NullOutput)
