# Design Patterns: Creational - Multiton

## Multiton Pattern - Simple Explanation

**SITUATION: YOU NEED MULTIPLE CONTROLLED INSTANCES**  
You have different configurations for different scenarios.

Initially, you have:

#### Java
```java
class Configuration {
    private static Configuration instance;
    
    private Configuration() {}
    
    public static Configuration getInstance() {
        if (instance == null) {
            instance = new Configuration();
        }
        return instance;
    }
}
``` 
A simple singleton.

**THEN THE REQUIREMENTS CHANGE**  
Now you need:
- Development config
- Production config
- Testing config
- Different database connections
- Multiple caches

You're stuck with:
```java
class Configuration {
    private static Configuration devInstance;
    private static Configuration prodInstance;
    private static Configuration testInstance;
    
    public static Configuration getDevInstance() {
        if (devInstance == null) {
            devInstance = new Configuration("dev");
        }
        return devInstance;
    }
    
    public static Configuration getProdInstance() {
        if (prodInstance == null) {
            prodInstance = new Configuration("prod");
        }
        return prodInstance;
    }
    // More getInstance methods...
}
``` 
Code explosion!

**WHAT'S THE PROBLEM?**  
- Multiple static instances
- Hard to manage different instances
- No unified approach
- Code duplication
- Inflexible design

**HOW MULTITON SAVES YOU**  
Create a registry of named instances.

You say: "Give me the instance for this key." 

And the Multiton:
- Manages multiple instances
- Controls creation
- Ensures uniqueness per key
- Provides flexible access
- Centralizes management

Now you just:
```java
Configuration config = Configuration.getInstance("dev");
// or
Configuration config = Configuration.getInstance("prod");
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java
```java
public class Multiton<K, V> {
    private static final Map<Class<?>, Map<Object, Object>> instances = 
        new ConcurrentHashMap<>();
    
    private final Supplier<V> factory;
    
    public Multiton(Supplier<V> factory) {
        this.factory = factory;
    }
    
    @SuppressWarnings("unchecked")
    public V getInstance(K key) {
        return (V) instances
            .computeIfAbsent(getClass(), k -> new ConcurrentHashMap<>())
            .computeIfAbsent(key, k -> factory.get());
    }
}

// Usage
class Configuration {
    private static final Multiton<String, Configuration> multiton = 
        new Multiton<>(Configuration::new);
    
    private Configuration() {}
    
    public static Configuration getInstance(String env) {
        return multiton.getInstance(env);
    }
}
```

#### TypeScript
```typescript
class Multiton<K, V> {
    private static instances = new Map<any, Map<any, any>>();
    
    constructor(private factory: () => V) {}
    
    getInstance(key: K): V {
        const classType = this.constructor;
        
        if (!Multiton.instances.has(classType)) {
            Multiton.instances.set(classType, new Map<K, V>());
        }
        
        const instanceMap = Multiton.instances.get(classType)!;
        
        if (!instanceMap.has(key)) {
            instanceMap.set(key, this.factory());
        }
        
        return instanceMap.get(key);
    }
}

// Usage
class Configuration {
    private static multiton = new Multiton(() => new Configuration());
    
    private constructor() {}
    
    static getInstance(env: string): Configuration {
        return this.multiton.getInstance(env);
    }
}
```

#### Python
```python
from typing import TypeVar, Generic, Dict, Callable, Any
from threading import Lock

K = TypeVar('K')
V = TypeVar('V')

class Multiton(Generic[K, V]):
    _instances: Dict[type, Dict[Any, Any]] = {}
    _lock = Lock()
    
    def __init__(self, factory: Callable[[], V]):
        self.factory = factory
    
    def get_instance(self, key: K) -> V:
        with self._lock:
            if self.__class__ not in self._instances:
                self._instances[self.__class__] = {}
            
            instances = self._instances[self.__class__]
            if key not in instances:
                instances[key] = self.factory()
            
            return instances[key]

# Usage
class Configuration:
    _multiton = Multiton(lambda: Configuration())
    
    def __init__(self):
        pass
    
    @classmethod
    def get_instance(cls, env: str) -> 'Configuration':
        return cls._multiton.get_instance(env)
```

#### Go
```go
package main

import (
    "sync"
)

type Multiton[K comparable, V any] struct {
    mu      sync.RWMutex
    factory func() V
    instances map[K]V
}

func NewMultiton[K comparable, V any](factory func() V) *Multiton[K, V] {
    return &Multiton[K, V]{
        factory:   factory,
        instances: make(map[K]V),
    }
}

func (m *Multiton[K, V]) GetInstance(key K) V {
    m.mu.RLock()
    if instance, exists := m.instances[key]; exists {
        m.mu.RUnlock()
        return instance
    }
    m.mu.RUnlock()
    
    m.mu.Lock()
    defer m.mu.Unlock()
    
    // Double-check after acquiring write lock
    if instance, exists := m.instances[key]; exists {
        return instance
    }
    
    instance := m.factory()
    m.instances[key] = instance
    return instance
}

// Usage
type Configuration struct {
    // Configuration fields
}

var (
    configMultiton = NewMultiton(func() *Configuration {
        return &Configuration{}
    })
)

func GetConfiguration(env string) *Configuration {
    return configMultiton.GetInstance(env)
}
```

**Key Differences:**
- Java uses ConcurrentHashMap for thread safety
- TypeScript uses nested Maps
- Python uses a class-level dictionary with locks
- Go implements with generics and RWMutex

**WHEN SHOULD YOU USE IT?**
- When you need controlled instances
- When instances vary by key
- When managing different configurations
- When implementing caches
- When resource pooling

**WHERE YOU'VE ALREADY SEEN IT**
- Database connection pools
- Configuration management
- Cache implementations
- Resource managers
- Plugin systems
