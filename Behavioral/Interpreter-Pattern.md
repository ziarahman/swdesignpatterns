# Design Patterns: Behavioral - Interpreter

## Interpreter Pattern - Simple Explanation

**SITUATION: YOU'RE PARSING EXPRESSIONS**  
You have a simple expression to evaluate.

Initially, you have:

#### Java
```java
class Calculator {
    int evaluate(String expression) {
        return Integer.parseInt(expression);
    }
}
``` 
All good.

**THEN THE EXPRESSIONS GET COMPLEX**  
Now you need to handle:
- Mathematical operations
- Logical expressions
- Custom functions
- Variables
- Nested expressions

You're stuck with:
```java
if (expression.contains("+")) {
    String[] parts = expression.split("\\+");
    return Integer.parseInt(parts[0]) + Integer.parseInt(parts[1]);
} else if (expression.contains("-")) {
    // More parsing logic...
}
``` 
Parser spaghetti!

**WHAT'S THE PROBLEM?**  
- Hard to add new operations
- Complex parsing logic
- No grammar structure
- Hard to maintain
- Limited extensibility

**HOW INTERPRETER SAVES YOU**  
Define a grammar for your language and create interpreters for each rule.

You say: "Here's my expression, interpret it." 

And the Interpreter:
- Breaks down complex expressions
- Follows grammar rules
- Evaluates step by step
- Handles nested structures
- Makes adding operations easy

Now you just:
```java
Expression expr = Parser.parse("5 + (10 * 2)");
int result = expr.interpret();
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java
```java
interface Expression {
    int interpret();
}

class NumberExpression implements Expression {
    private final int number;
    
    public NumberExpression(int number) {
        this.number = number;
    }
    
    @Override
    public int interpret() {
        return number;
    }
}

class AddExpression implements Expression {
    private final Expression left;
    private final Expression right;
    
    public AddExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public int interpret() {
        return left.interpret() + right.interpret();
    }
}

class MultiplyExpression implements Expression {
    private final Expression left;
    private final Expression right;
    
    public MultiplyExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public int interpret() {
        return left.interpret() * right.interpret();
    }
}

// Usage
class Parser {
    public static Expression parse(String expression) {
        // Parsing logic to build expression tree
        return new AddExpression(
            new NumberExpression(5),
            new MultiplyExpression(
                new NumberExpression(10),
                new NumberExpression(2)
            )
        );
    }
}
```

#### TypeScript
```typescript
interface Expression {
    interpret(): number;
}

class NumberExpression implements Expression {
    constructor(private value: number) {}
    
    interpret(): number {
        return this.value;
    }
}

class AddExpression implements Expression {
    constructor(
        private left: Expression,
        private right: Expression
    ) {}
    
    interpret(): number {
        return this.left.interpret() + this.right.interpret();
    }
}

class MultiplyExpression implements Expression {
    constructor(
        private left: Expression,
        private right: Expression
    ) {}
    
    interpret(): number {
        return this.left.interpret() * this.right.interpret();
    }
}

// Usage
class Parser {
    static parse(expression: string): Expression {
        // Parsing logic to build expression tree
        return new AddExpression(
            new NumberExpression(5),
            new MultiplyExpression(
                new NumberExpression(10),
                new NumberExpression(2)
            )
        );
    }
}
```

#### Python
```python
from abc import ABC, abstractmethod

class Expression(ABC):
    @abstractmethod
    def interpret(self) -> int:
        pass

class NumberExpression(Expression):
    def __init__(self, value: int):
        self.value = value
    
    def interpret(self) -> int:
        return self.value

class AddExpression(Expression):
    def __init__(self, left: Expression, right: Expression):
        self.left = left
        self.right = right
    
    def interpret(self) -> int:
        return self.left.interpret() + self.right.interpret()

class MultiplyExpression(Expression):
    def __init__(self, left: Expression, right: Expression):
        self.left = left
        self.right = right
    
    def interpret(self) -> int:
        return self.left.interpret() * self.right.interpret()

# Usage
class Parser:
    @staticmethod
    def parse(expression: str) -> Expression:
        # Parsing logic to build expression tree
        return AddExpression(
            NumberExpression(5),
            MultiplyExpression(
                NumberExpression(10),
                NumberExpression(2)
            )
        )
```

#### Go
```go
package main

type Expression interface {
    Interpret() int
}

type NumberExpression struct {
    value int
}

func NewNumberExpression(value int) *NumberExpression {
    return &NumberExpression{value: value}
}

func (n *NumberExpression) Interpret() int {
    return n.value
}

type AddExpression struct {
    left  Expression
    right Expression
}

func NewAddExpression(left, right Expression) *AddExpression {
    return &AddExpression{left: left, right: right}
}

func (a *AddExpression) Interpret() int {
    return a.left.Interpret() + a.right.Interpret()
}

type MultiplyExpression struct {
    left  Expression
    right Expression
}

func NewMultiplyExpression(left, right Expression) *MultiplyExpression {
    return &MultiplyExpression{left: left, right: right}
}

func (m *MultiplyExpression) Interpret() int {
    return m.left.Interpret() * m.right.Interpret()
}

// Usage
type Parser struct{}

func (p *Parser) Parse(expression string) Expression {
    // Parsing logic to build expression tree
    return NewAddExpression(
        NewNumberExpression(5),
        NewMultiplyExpression(
            NewNumberExpression(10),
            NewNumberExpression(2),
        ),
    )
}
```

**Key Differences:**
- Java uses interfaces for expressions
- TypeScript implements with strict typing
- Python uses abstract base classes
- Go uses interfaces and structs

**WHEN SHOULD YOU USE IT?**
- When parsing complex expressions
- When implementing a simple language
- When evaluating business rules
- When building query languages
- When processing structured text

**WHERE YOU'VE ALREADY SEEN IT**
- SQL parsers
- Regular expressions
- Mathematical expression evaluators
- Rule engines
- Template engines
