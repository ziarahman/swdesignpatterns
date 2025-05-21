# Design Patterns: Structural - Flyweight

## Flyweight Pattern - Simple Explanation

**SITUATION: YOU’RE CREATING MANY SIMILAR OBJECTS**  
You’re building a text editor that renders characters. Each character has a font, size, and color. 

Initially, you create: 

#### Java
```java
new Character('A', "Arial", 12, "black");
``` 

All good.

**THEN THE DOCUMENT GROWS**  
Now you have a million characters. Many share the same font, size, and color (e.g., all 'A's in Arial 12 black). 

You’re stuck creating: 
```java
new Character('A', "Arial", 12, "black");
``` 
A million times. 

It’s wasting memory.

**WHAT’S THE PROBLEM?**  
- You’re creating too many objects with shared properties.  
- Each object stores the same font, size, and color data—wasting memory.  
- Performance degrades as the number of objects grows.  
- It’s inefficient for large-scale applications.

**HOW FLYWEIGHT SAVES YOU**  
Instead of storing shared data in each object, you create a `Flyweight` (e.g., `CharacterStyle`) to store the shared properties (font, size, color). You reuse the same `CharacterStyle` for all characters with those properties. The character only stores its unique data (e.g., the letter 'A'). 

You say: “Hey Flyweight, give me the style for Arial 12 black.” 

Now you just: 
```java
CharacterStyle style = styleFactory.getStyle("Arial", 12, "black"); 
Character c = new Character('A', style);
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
class CharacterStyle {  
    private String font;  
    private int size;  
    private String color;  
    public CharacterStyle(String font, int size, String color) {  
        this.font = font;  
        this.size = size;  
        this.color = color;  
    }  
}  
class CharacterStyleFactory {  
    private Map<String, CharacterStyle> styles = new HashMap<>();  
    public CharacterStyle getStyle(String font, int size, String color) {  
        String key = font + "-" + size + "-" + color;  
        return styles.computeIfAbsent(key, k -> new CharacterStyle(font, size, color));  
    }  
}  
class Character {  
    private char charValue;  
    private CharacterStyle style;  
    public Character(char charValue, CharacterStyle style) {  
        this.charValue = charValue;  
        this.style = style;  
    }  
}  
// Usage  
CharacterStyleFactory factory = new CharacterStyleFactory();  
CharacterStyle style = factory.getStyle("Arial", 12, "black");  
Character c = new Character('A', style);  
```

#### TypeScript  
```typescript  
class CharacterStyle {  
    constructor(public font: string, public size: number, public color: string) {}  
}  
class CharacterStyleFactory {  
    private styles: { [key: string]: CharacterStyle } = {};  
    getStyle(font: string, size: number, color: string): CharacterStyle {  
        const key = `${font}-${size}-${color}`;  
        if (!this.styles[key]) {  
            this.styles[key] = new CharacterStyle(font, size, color);  
        }  
        return this.styles[key];  
    }  
}  
class Character {  
    constructor(public char: string, public style: CharacterStyle) {}  
}  
// Usage  
const factory = new CharacterStyleFactory();  
const style = factory.getStyle("Arial", 12, "black");  
const c = new Character('A', style);  
```

#### Python  
```python  
class CharacterStyle:  
    def __init__(self, font, size, color):  
        self.font, self.size, self.color = font, size, color  
class CharacterStyleFactory:  
    def __init__(self):  
        self.styles = {}  
    def get_style(self, font, size, color):  
        key = (font, size, color)  
        if key not in self.styles:  
            self.styles[key] = CharacterStyle(font, size, color)  
        return self.styles[key]  
class Character:  
    def __init__(self, char, style):  
        self.char, self.style = char, style  
# Usage  
factory = CharacterStyleFactory()  
style = factory.get_style("Arial", 12, "black")  
c = Character('A', style)  
```

#### Go  
```go  
package main  
type CharacterStyle struct {  
    Font, Color string  
    Size int  
}  
type CharacterStyleFactory struct {  
    styles map[string]*CharacterStyle  
}  
func (f *CharacterStyleFactory) GetStyle(font string, size int, color string) *CharacterStyle {  
    key := fmt.Sprintf("%s-%d-%s", font, size, color)  
    if style, exists := f.styles[key]; exists {  
        return style  
    }  
    style := &CharacterStyle{font, size, color}  
    f.styles[key] = style  
    return style  
}  
type Character struct {  
    Char string  
    Style *CharacterStyle  
}  
// Usage  
func main() {  
    factory := &CharacterStyleFactory{styles: make(map[string]*CharacterStyle)}  
    style := factory.GetStyle("Arial", 12, "black")  
    c := &Character{"A", style}  
}  
```

**Key Differences:**  
- Java and TypeScript use hash maps for style caching, ensuring type safety.  
- Python uses tuples as dictionary keys for a simple caching mechanism.  
- Go uses a map for caching, focusing on efficiency.  
- All reduce memory usage by sharing common data.

**WHEN SHOULD YOU USE IT?**  
- When you have many objects with shared properties.  
- When memory usage is a concern.  
- When the shared data can be separated from unique data.

**WHERE YOU’VE ALREADY SEEN IT**  
`String` interning in Java (reuses identical strings).  
`Glyphs` in text rendering (shares font data for characters).