# Design Patterns Collection

This repository contains a comprehensive collection of software design patterns, organized by categories. Each pattern includes detailed explanations, real-world scenarios, and implementations in multiple programming languages.

## Architectural Patterns
- [MVC (Model-View-Controller)](Architectural/MVC-Pattern.md) - Separates application logic into Model, View, and Controller components
- [MVVM (Model-View-ViewModel)](Architectural/MVVM-Pattern.md) - Separates UI from business logic using a ViewModel
- [Repository](Architectural/Repository-Pattern.md) - Abstracts data persistence from business logic
- [Unit of Work](Architectural/UnitOfWork-Pattern.md) - Maintains a list of business objects affected by a transaction

## Behavioral Patterns
- [Chain of Responsibility](Behavioral/ChainOfResponsibility-Pattern.md) - Passes requests along a chain of handlers
- [Command](Behavioral/Command-Pattern.md) - Encapsulates a request as an object
- [Interpreter](Behavioral/Interpreter-Pattern.md) - Defines a grammar for interpreting expressions
- [Iterator](Behavioral/Iterator-Pattern.md) - Provides sequential access to elements
- [Mediator](Behavioral/Mediator-Pattern.md) - Reduces coupling between components by centralizing communication
- [Memento](Behavioral/Memento-Pattern.md) - Captures and restores an object's internal state
- [Null Object](Behavioral/NullObject-Pattern.md) - Provides default behavior for null objects
- [Observer](Behavioral/Observer-Pattern.md) - Defines one-to-many dependency between objects
- [Pipeline](Behavioral/Pipeline-Pattern.md) - Processes data through a series of operations
- [Specification](Behavioral/Specification-Pattern.md) - Encapsulates business rules
- [State](Behavioral/State-Pattern.md) - Allows object behavior to change based on internal state
- [Strategy](Behavioral/Strategy-Pattern.md) - Defines family of algorithms that are interchangeable
- [Template Method](Behavioral/TemplateMethod-Pattern.md) - Defines skeleton of an algorithm, letting subclasses override specific steps
- [Visitor](Behavioral/Visitor-Pattern.md) - Separates algorithms from the objects they operate on

## Cloud Patterns
- [Ambassador](Cloud/Ambassador-Pattern.md) - Provides helper services for the main service
- [Anti-Corruption Layer](Cloud/Anti-Corruption-Layer-Pattern.md) - Prevents incompatible models from corrupting each other
- [Sidecar](Cloud/Sidecar-Pattern.md) - Deploys components of an application as separate containers

## Concurrency Patterns
- [Active Object](Concurrency/ActiveObject-Pattern.md) - Decouples method execution from method invocation
- [Barrier](Concurrency/Barrier-Pattern.md) - Synchronizes multiple threads at a common point
- [Future](Concurrency/Future-Pattern.md) - Represents result of an asynchronous computation
- [Monitor](Concurrency/Monitor-Pattern.md) - Synchronizes access to a shared resource
- [Producer-Consumer](Concurrency/ProducerConsumer-Pattern.md) - Coordinates production and consumption of shared resources
- [Reactor](Concurrency/Reactor-Pattern.md) - Handles service requests delivered concurrently
- [Thread Pool](Concurrency/ThreadPool-Pattern.md) - Manages a pool of worker threads

## Creational Patterns
- [Abstract Factory](Creational/AbstractFactory-Pattern.md) - Creates families of related objects
- [Builder](Creational/Builder-Pattern.md) - Separates complex object construction from its representation
- [Factory](Creational/Factory-Pattern.md) - Creates objects without exposing instantiation logic
- [Multiton](Creational/Multiton-Pattern.md) - Controls creation of a fixed number of instances
- [Object Pool](Creational/ObjectPool-Pattern.md) - Improves performance by reusing objects
- [Prototype](Creational/Prototype-Pattern.md) - Creates new objects by cloning an existing instance
- [Singleton](Creational/Singleton-Pattern.md) - Ensures a class has only one instance

## Domain Patterns
- [Aggregate](Domain/Aggregate-Pattern.md) - Clusters domain objects into a single unit
- [CQRS](Domain/CQRS-Pattern.md) - Separates read and write operations
- [Domain Event](Domain/Domain-Event-Pattern.md) - Captures and communicates domain changes
- [Event Sourcing](Domain/Event-Sourcing-Pattern.md) - Stores state as a sequence of events

## Enterprise Patterns
- [DAO (Data Access Object)](Enterprise/DAO-Pattern.md) - Separates data access logic from business logic
- [DTO (Data Transfer Object)](Enterprise/DTO-Pattern.md) - Transfers data between subsystems
- [Service Locator](Enterprise/ServiceLocator-Pattern.md) - Centralizes service object lookup

## Integration Patterns
- [Bulkhead](Integration/Bulkhead-Pattern.md) - Isolates elements of an application into pools
- [Circuit Breaker](Integration/Circuit-Breaker-Pattern.md) - Prevents cascading failures
- [Gateway](Integration/Gateway-Pattern.md) - Provides unified interface to external services

## Security Patterns
- [Authentication](Security/Authentication-Pattern.md) - Verifies identity of users and systems
- [Authorization](Security/Authorization-Pattern.md) - Controls access to resources and operations

## Structural Patterns
- [Adapter](Structural/Adapter-Pattern.md) - Makes incompatible interfaces compatible
- [Bridge](Structural/Bridge-Patter.md) - Separates abstraction from implementation
- [Composite](Structural/Composite-Pattern.md) - Treats groups of objects as a single object
- [Decorator](Structural/Decorator-Patter.md) - Adds behavior to objects dynamically
- [Facade](Structural/Facade-Pattern.md) - Provides simplified interface to complex subsystem
- [Flyweight](Structural/Flyweight-Pattern.md) - Minimizes memory usage by sharing data
- [Proxy](Structural/Proxy-Pattern.md) - Controls access to an object

## Pattern Markdown File
Each pattern documentation includes:
- Simple explanation with analogies
- Real-world scenarios
- Implementation examples in multiple languages
- Usage guidelines and best practices
- Key implementation differences

## CREDITS

This design patterns collection is maintained and documented with contributions from:

### Primary Contributors
- Zia (Project Creator & Maintainer) + Vibe Coding with GH Copilot & other AI Agents 

### References & Inspiration
- "Design Patterns: Elements of Reusable Object-Oriented Software" by Gang of Four
- Martin Fowler's "Patterns of Enterprise Application Architecture"
- Cloud Native Computing Foundation (CNCF) Patterns
- Microsoft's Cloud Design Patterns
- Domain-Driven Design Community
- Inspired by posts made on Design Patterns by https://x.com/theskilledcoder

### License
This documentation is licensed under MIT License. Feel free to use, modify, and distribute with proper attribution.

