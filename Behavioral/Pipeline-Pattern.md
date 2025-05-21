# Design Patterns: Behavioral - Pipeline

## Pipeline Pattern - Simple Explanation

**SITUATION: YOU'RE PROCESSING DATA IN STEPS**  
You have data that needs multiple processing steps.

Initially, you have:

#### TypeScript
```typescript
class DataProcessor {
    process(data: string): string {
        const cleaned = data.trim();
        const normalized = cleaned.toLowerCase();
        const filtered = normalized.replace(/[^a-z]/g, '');
        return filtered;
    }
}
``` 
All good.

**THEN THE PROCESSING GROWS**  
Now you need:
- More processing steps
- Different combinations
- Reusable steps
- Complex workflows
- Error handling at each step

You're stuck with:
```typescript
class DataProcessor {
    process(data: string): string {
        try {
            const cleaned = this.clean(data);
            try {
                const normalized = this.normalize(cleaned);
                try {
                    const validated = this.validate(normalized);
                    try {
                        const transformed = this.transform(validated);
                        return transformed;
                    } catch (error) {
                        // Handle transform error
                    }
                } catch (error) {
                    // Handle validation error
                }
            } catch (error) {
                // Handle normalization error
            }
        } catch (error) {
            // Handle cleaning error
        }
    }
}
``` 
Nested nightmare!

**WHAT'S THE PROBLEM?**  
- Steps are tightly coupled
- Hard to reorder steps
- Error handling is messy
- Code is hard to maintain
- Steps can't be reused

**HOW PIPELINE SAVES YOU**  
Create a series of independent steps that can be chained together.

You say: "Process this data through these steps." 

And the Pipeline:
- Manages step sequence
- Handles data flow
- Processes step by step
- Manages errors
- Makes steps reusable

Now you just:
```typescript
const pipeline = new Pipeline<string>()
    .pipe(clean)
    .pipe(normalize)
    .pipe(validate)
    .pipe(transform);

const result = await pipeline.execute(data);
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### TypeScript
```typescript
type PipeFunction<T> = (data: T) => Promise<T> | T;

class Pipeline<T> {
    private steps: PipeFunction<T>[] = [];
    
    pipe(fn: PipeFunction<T>): Pipeline<T> {
        this.steps.push(fn);
        return this;
    }
    
    async execute(initialData: T): Promise<T> {
        return this.steps.reduce(
            async (promise, step) => step(await promise),
            Promise.resolve(initialData)
        );
    }
}

// Usage
const clean = (data: string) => data.trim();
const normalize = (data: string) => data.toLowerCase();
const validate = (data: string) => {
    if (data.length === 0) throw new Error('Empty data');
    return data;
};
const transform = (data: string) => data.replace(/[^a-z]/g, '');

const pipeline = new Pipeline<string>()
    .pipe(clean)
    .pipe(normalize)
    .pipe(validate)
    .pipe(transform);

const result = await pipeline.execute(' Hello, World! ');
```

#### Java
```java
import java.util.ArrayList;
import java.util.List;
import java.util.function.Function;
import java.util.stream.Stream;

public class Pipeline<T> {
    private final List<Function<T, T>> steps = new ArrayList<>();
    
    public Pipeline<T> pipe(Function<T, T> step) {
        steps.add(step);
        return this;
    }
    
    public T execute(T initialData) {
        return steps.stream()
            .reduce(Function.identity(), Function::andThen)
            .apply(initialData);
    }
}

// Usage with Stream API
class DataProcessor {
    public static void main(String[] args) {
        Pipeline<String> pipeline = new Pipeline<String>()
            .pipe(String::trim)
            .pipe(String::toLowerCase)
            .pipe(s -> s.replaceAll("[^a-z]", ""));
        
        String result = pipeline.execute(" Hello, World! ");
    }
}
```

#### Python
```python
from typing import TypeVar, Callable, List
from functools import reduce

T = TypeVar('T')

class Pipeline:
    def __init__(self):
        self.steps: List[Callable[[T], T]] = []
    
    def pipe(self, step: Callable[[T], T]) -> 'Pipeline[T]':
        self.steps.append(step)
        return self
    
    def execute(self, initial_data: T) -> T:
        return reduce(
            lambda data, step: step(data),
            self.steps,
            initial_data
        )

# Usage
def clean(data: str) -> str:
    return data.strip()

def normalize(data: str) -> str:
    return data.lower()

def validate(data: str) -> str:
    if not data:
        raise ValueError("Empty data")
    return data

def transform(data: str) -> str:
    return ''.join(c for c in data if c.isalpha())

pipeline = Pipeline()
pipeline.pipe(clean) \
        .pipe(normalize) \
        .pipe(validate) \
        .pipe(transform)

result = pipeline.execute(" Hello, World! ")
```

#### Go
```go
package main

import (
    "strings"
)

type PipeFunc[T any] func(T) T

type Pipeline[T any] struct {
    steps []PipeFunc[T]
}

func NewPipeline[T any]() *Pipeline[T] {
    return &Pipeline[T]{
        steps: make([]PipeFunc[T], 0),
    }
}

func (p *Pipeline[T]) Pipe(fn PipeFunc[T]) *Pipeline[T] {
    p.steps = append(p.steps, fn)
    return p
}

func (p *Pipeline[T]) Execute(initialData T) T {
    result := initialData
    for _, step := range p.steps {
        result = step(result)
    }
    return result
}

// Usage
func clean(data string) string {
    return strings.TrimSpace(data)
}

func normalize(data string) string {
    return strings.ToLower(data)
}

func validate(data string) string {
    if len(data) == 0 {
        panic("Empty data")
    }
    return data
}

func transform(data string) string {
    var result strings.Builder
    for _, c := range data {
        if (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z') {
            result.WriteRune(c)
        }
    }
    return result.String()
}

func main() {
    pipeline := NewPipeline[string]().
        Pipe(clean).
        Pipe(normalize).
        Pipe(validate).
        Pipe(transform)
    
    result := pipeline.Execute(" Hello, World! ")
}
```

**Key Differences:**
- TypeScript uses async/await for asynchronous steps
- Java leverages Stream API and Function composition
- Python uses reduce and type hints
- Go implements with generics and method chaining

**WHEN SHOULD YOU USE IT?**
- When processing data in steps
- When steps need to be reusable
- When order matters
- When handling data transformations
- When building data processing workflows

**WHERE YOU'VE ALREADY SEEN IT**
- ETL processes
- Image processing pipelines
- Build systems
- Data processing frameworks
- Middleware chains
