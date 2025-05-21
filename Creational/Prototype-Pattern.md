# Design Patterns: Simple Explanations

## Prototype Pattern

**SITUATION: YOU’RE CREATING OBJECTS FROM A TEMPLATE**  
You’re building a game with enemies like `Goblin`. Initially, you create each one: 

#### Java
```java
Goblin goblin = new Goblin(100, 10, "axe");
``` 
All good.

**THEN THE GAME GROWS**  
Now you need 1000 goblins with slight variations. You’re stuck writing: 

#### Java
```java
Goblin g1 = new Goblin(100, 10, "axe"); 
Goblin g2 = new Goblin(100, 10, "axe");
``` 
Everywhere. 

It’s slow and repetitive.

**WHAT’S THE PROBLEM?**  
- Creating each object from scratch is expensive and slow.  
- Most objects share the same base properties with minor tweaks.  
- You’re wasting resources on redundant creation.  
- It’s hard to manage variations consistently.

**HOW PROTOTYPE SAVES YOU**  
Instead of creating from scratch, you create a `Goblin` prototype and clone it. You say: “Hey, give me a copy of this `Goblin`.” And the Prototype:  
- Clones the base object quickly.  
- Lets you tweak the copy as needed.  
- Saves time and resources.  

Now you just: 

#### Java
```java
Goblin prototype = new Goblin(100, 10, "axe"); 
Goblin g1 = prototype.clone();
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
interface Cloneable {  
    Object clone();  
}  
class Goblin implements Cloneable {  
    private int health, attack;  
    private String weapon;  
    public Goblin(int health, int attack, String weapon) {  
        this.health = health;  
        this.attack = attack;  
        this.weapon = weapon;  
    }  
    public Goblin clone() {  
        return new Goblin(health, attack, weapon);  
    }  
}  
// Usage  
Goblin prototype = new Goblin(100, 10, "axe");  
Goblin g1 = prototype.clone();  
```

#### TypeScript  
```typescript  
interface Cloneable {  
    clone(): this;  
}  
class Goblin implements Cloneable {  
    constructor(public health: number, public attack: number, public weapon: string) {}  
    clone(): this {  
        return Object.create(this);  
    }  
}  
// Usage  
const prototype = new Goblin(100, 10, "axe");  
const g1 = prototype.clone();  
```

#### Python  
```python  
import copy  
class Goblin:  
    def __init__(self, health, attack, weapon):  
        self.health, self.attack, self.weapon = health, attack, weapon  
    def clone(self):  
        return copy.deepcopy(self)  
# Usage  
prototype = Goblin(100, 10, "axe")  
g1 = prototype.clone()  
```

#### Go  
```go  
package main  
import "encoding/json"  
type Goblin struct {  
    Health, Attack int  
    Weapon string  
}  
func (g *Goblin) Clone() *Goblin {  
    data, _ := json.Marshal(g)  
    var clone Goblin  
    json.Unmarshal(data, &clone)  
    return &clone  
}  
// Usage  
func main() {  
    prototype := &Goblin{100, 10, "axe"}  
    g1 := prototype.Clone()  
}  
```

**Key Differences:**  
- Java often uses a `Cloneable` interface or deep copying methods.  
- TypeScript leverages `Object.create()` for prototyping, fitting JavaScript’s prototype-based nature.  
- Python uses the `copy` module for deep copying, which is built-in and straightforward.  
- Go lacks a native clone mechanism, so serialization (e.g., JSON) is often used for deep copies.  
- All achieve efficient object creation by cloning.

**WHEN SHOULD YOU USE IT?**  
- When creating objects from scratch is expensive.  
- When objects are similar but have slight variations.  
- When you want to avoid repetitive initialization logic.

**WHERE YOU’VE ALREADY SEEN IT**  
`Object.clone()` in Java (for cloning objects).  
`Prototype` in JavaScript (for object creation).