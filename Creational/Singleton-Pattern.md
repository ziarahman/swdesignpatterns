# Design Patterns: Creational

## Singleton Pattern - Simple Explanations

**SITUATION: YOU NEED ONE AND ONLY ONE INSTANCE**  
You’re building a feature that needs a single, shared resource—like a configuration manager or a logger. Initially, you create it like: 

#### Java
```java
ConfigManager config = new ConfigManager();
```

All good.

**THEN THE APP GROWS**  
Now you have:  
- Multiple classes accessing the config.  
- Each creating its own `ConfigManager` instance.  
You’re stuck with: 
#### Java
```java
ConfigManager config1 = new ConfigManager(); 
ConfigManager config2 = new ConfigManager(); 
```
Everywhere.

They’re all separate instances—wasting memory and causing inconsistency.

**WHAT’S THE PROBLEM?**  
- You only want *one* instance, but you’re creating many.  
- Different parts of the app might overwrite shared data.  
- It’s hard to control access and ensure consistency.  
- Testing becomes tricky with multiple instances floating around.

**HOW SINGLETON SAVES YOU**  
Instead of creating multiple instances, you use a Singleton pattern. You say: “Hey, give me the *single* instance of `ConfigManager`.” And the Singleton:  
- Ensures only one instance exists.  
- Provides global access to that instance.  
- Keeps your app consistent and memory-efficient.  

Now you don’t care how it’s created—you just get the same instance every time: 
#### Java
```java
ConfigManager config = ConfigManager.getInstance();
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### TypeScript
```typescript
class ConfigManager {
    private static instance: ConfigManager;
    private constructor() { }

    public static getInstance(): ConfigManager {
        if (!ConfigManager.instance) {
            ConfigManager.instance = new ConfigManager();
        }
        return ConfigManager.instance;
    }
}

// Usage
const config = ConfigManager.getInstance();
```

#### Python
```python
class ConfigManager:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    # Alternative using decorator
    @classmethod
    def getInstance(cls):
        if cls._instance is None:
            cls._instance = cls()
        return cls._instance

# Usage
config = ConfigManager()  # or ConfigManager.getInstance()
```

#### Go
```go
package main

import "sync"

type ConfigManager struct{}

var instance *ConfigManager
var once sync.Once

func GetInstance() *ConfigManager {
    once.Do(func() {
        instance = &ConfigManager{}
    })
    return instance
}

// Usage
func main() {
    config := GetInstance()
}
```

**Key Differences:**
- TypeScript uses private constructors and static methods similar to Java
- Python can use `__new__` or class methods with a more pythonic approach
- Go implements thread-safety using `sync.Once` out of the box
- All achieve the same goal: single instance, global access

**WHEN SHOULD YOU USE IT?**  
- When you need exactly *one* instance of a class (e.g., a logger, config manager).  
- When you want a globally accessible point without passing instances around.  
- When the instance doesn’t need to change or be replaced.

**WHERE YOU’VE ALREADY SEEN IT**  
`Runtime.getRuntime()` in Java (for JVM runtime environment).  
`getApplicationContext()` in Android (for app-wide context).