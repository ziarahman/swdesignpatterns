# Design Patterns: Behavioral - Iterator

## Iterator Pattern - Simple Explanation

**SITUATION: YOU'RE ACCESSING COLLECTIONS**  
You're working with a list of items. 

Initially, you have: 

#### Java
```java
for (int i = 0; i < array.length; i++) {
    process(array[i]);
}
``` 
All good.

**THEN THE COLLECTIONS VARY**  
Now you have:  
- Arrays
- Lists
- Trees
- Custom Collections

Each needs different access:
```java
// Array
for (int i = 0; i < array.length; i++) { ... }

// List
while (list.hasNext()) { ... }

// Tree
traverseInOrder(root); 
``` 
Gets messy.

**WHAT'S THE PROBLEM?**  
- Different collections need different traversal code.
- Client code needs to know collection internals.
- Hard to change collection type without changing code.
- Can't easily switch traversal strategies.

**HOW ITERATOR SAVES YOU**  
Define a standard way to traverse any collection.

You say: "Just give me an iterator, I'll handle the rest." 

And the Iterator:  
- Hides collection internals.
- Provides uniform access.
- Allows multiple traversal strategies.
- Makes collection swapping easy.

Now you just: 
```java
Iterator<Item> iterator = collection.iterator();
while (iterator.hasNext()) {
    process(iterator.next());
}
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
interface Iterator<T> {
    boolean hasNext();
    T next();
}

interface Collection<T> {
    Iterator<T> iterator();
}

class ArrayIterator<T> implements Iterator<T> {
    private T[] array;
    private int position = 0;
    
    public ArrayIterator(T[] array) {
        this.array = array;
    }
    
    public boolean hasNext() {
        return position < array.length;
    }
    
    public T next() {
        if (this.hasNext()) {
            return array[position++];
        }
        throw new IndexOutOfBoundsException();
    }
}

class Array<T> implements Collection<T> {
    private T[] items;
    
    public Array(T[] items) {
        this.items = items;
    }
    
    public Iterator<T> iterator() {
        return new ArrayIterator<T>(items);
    }
}
```

#### TypeScript  
```typescript  
interface Iterator<T> {
    hasNext(): boolean;
    next(): T;
    current(): T;
}

interface Collection<T> {
    createIterator(): Iterator<T>;
}

class ArrayIterator<T> implements Iterator<T> {
    private collection: T[];
    private position: number = 0;

    constructor(collection: T[]) {
        this.collection = collection;
    }

    public hasNext(): boolean {
        return this.position < this.collection.length;
    }

    public next(): T {
        if (this.hasNext()) {
            return this.collection[this.position++];
        }
        throw new Error('No more elements');
    }

    public current(): T {
        return this.collection[this.position];
    }
}

class NumberCollection implements Collection<number> {
    private items: number[] = [];

    public addItem(item: number): void {
        this.items.push(item);
    }

    public createIterator(): Iterator<number> {
        return new ArrayIterator(this.items);
    }
}
```

#### Python  
```python  
from abc import ABC, abstractmethod
from typing import Generic, TypeVar, List

T = TypeVar('T')

class Iterator(ABC, Generic[T]):
    @abstractmethod
    def has_next(self) -> bool:
        pass
    
    @abstractmethod
    def next(self) -> T:
        pass

class Collection(ABC, Generic[T]):
    @abstractmethod
    def create_iterator(self) -> Iterator[T]:
        pass

class ArrayIterator(Iterator[T]):
    def __init__(self, collection: List[T]):
        self._collection = collection
        self._position = 0
    
    def has_next(self) -> bool:
        return self._position < len(self._collection)
    
    def next(self) -> T:
        if self.has_next():
            item = self._collection[self._position]
            self._position += 1
            return item
        raise IndexError("No more elements")

class NumberCollection(Collection[int]):
    def __init__(self):
        self._items: List[int] = []
    
    def add_item(self, item: int):
        self._items.append(item)
    
    def create_iterator(self) -> Iterator[int]:
        return ArrayIterator(self._items)
```

#### Go  
```go  
package main

import "fmt"

// Iterator interface
type Iterator interface {
    HasNext() bool
    Next() interface{}
}

// Collection interface
type Collection interface {
    CreateIterator() Iterator
}

// Array iterator implementation
type ArrayIterator struct {
    array     []interface{}
    position  int
}

func (i *ArrayIterator) HasNext() bool {
    return i.position < len(i.array)
}

func (i *ArrayIterator) Next() interface{} {
    if i.HasNext() {
        item := i.array[i.position]
        i.position++
        return item
    }
    return nil
}

// Array collection
type Array struct {
    items []interface{}
}

func (a *Array) Add(item interface{}) {
    a.items = append(a.items, item)
}

func (a *Array) CreateIterator() Iterator {
    return &ArrayIterator{
        array: a.items,
    }
}

// Usage example
func main() {
    array := &Array{}
    array.Add("A")
    array.Add("B")
    array.Add("C")

    iterator := array.CreateIterator()
    for iterator.HasNext() {
        fmt.Println(iterator.Next())
    }
}
```

**Key Differences:**  
- Java uses generic interfaces for type safety.
- TypeScript adds current() method for convenience.
- Python uses ABC and type hints for clarity.
- Go uses interface{} for flexibility but loses type safety.

**WHEN SHOULD YOU USE IT?**  
- When you need to access elements sequentially.
- When you want to hide collection implementation details.
- When you need multiple ways to traverse a collection.
- When you want consistent traversal interface across collections.

**WHERE YOU'VE ALREADY SEEN IT**  
- Java Collections Framework
- Python's iterators and generators
- JavaScript's Symbol.iterator
- Database cursors
