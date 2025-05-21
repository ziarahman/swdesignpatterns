# Design Patterns: Enterprise - Service Locator

## Service Locator Pattern - Simple Explanation

**SITUATION: YOU NEED SERVICES**  
You have components that need to find and use services.

Initially, you have:

```java
class OrderProcessor {
    private final EmailService emailService;
    private final PaymentService paymentService;
    private final ShippingService shippingService;
    
    public OrderProcessor() {
        this.emailService = new EmailService();
        this.paymentService = new PaymentService();
        this.shippingService = new ShippingService();
    }
}
```

**THEN IT GETS COMPLEX**  
- Services have dependencies
- Different implementations needed
- Testing becomes difficult
- Configuration changes
- Runtime service discovery

**HOW SERVICE LOCATOR HELPS**  
Create a central registry for services.

You say: "I need a service" and the locator finds it for you.

The Service Locator:
- Maintains service registry
- Manages lifecycles
- Provides lookup mechanism
- Handles configuration
- Supports testing

Now you have:
```java
class OrderProcessor {
    private final EmailService emailService;
    private final PaymentService paymentService;
    private final ShippingService shippingService;
    
    public OrderProcessor() {
        this.emailService = ServiceLocator.getService(EmailService.class);
        this.paymentService = ServiceLocator.getService(PaymentService.class);
        this.shippingService = ServiceLocator.getService(ShippingService.class);
    }
}
```

## Implementation Examples

#### Java
```java
// Service interfaces
public interface EmailService {
    void sendEmail(String to, String subject, String body);
}

public interface PaymentService {
    boolean processPayment(Order order);
}

// Service Locator
public class ServiceLocator {
    private static final Map<Class<?>, Object> services = new ConcurrentHashMap<>();
    
    public static <T> void register(Class<T> serviceType, T implementation) {
        services.put(serviceType, implementation);
    }
    
    @SuppressWarnings("unchecked")
    public static <T> T getService(Class<T> serviceType) {
        T service = (T) services.get(serviceType);
        if (service == null) {
            throw new ServiceNotFoundException(serviceType);
        }
        return service;
    }
    
    public static void clear() {
        services.clear();
    }
}

// Usage
public class Application {
    public static void main(String[] args) {
        // Register services
        ServiceLocator.register(EmailService.class, new SmtpEmailService());
        ServiceLocator.register(PaymentService.class, new StripePaymentService());
        
        // Use services
        OrderProcessor processor = new OrderProcessor();
        processor.process(order);
    }
}
```

#### TypeScript
```typescript
// Service interfaces
interface EmailService {
    sendEmail(to: string, subject: string, body: string): void;
}

interface PaymentService {
    processPayment(order: Order): Promise<boolean>;
}

// Service Locator
class ServiceLocator {
    private static services = new Map<Function, any>();
    
    static register<T>(serviceType: Function, implementation: T): void {
        ServiceLocator.services.set(serviceType, implementation);
    }
    
    static getService<T>(serviceType: Function): T {
        const service = ServiceLocator.services.get(serviceType);
        if (!service) {
            throw new Error(`Service not found: ${serviceType.name}`);
        }
        return service;
    }
    
    static clear(): void {
        ServiceLocator.services.clear();
    }
}

// Usage
class OrderProcessor {
    private emailService: EmailService;
    private paymentService: PaymentService;
    
    constructor() {
        this.emailService = ServiceLocator.getService<EmailService>(EmailService);
        this.paymentService = ServiceLocator.getService<PaymentService>(PaymentService);
    }
    
    async process(order: Order): Promise<void> {
        if (await this.paymentService.processPayment(order)) {
            this.emailService.sendEmail(
                order.email,
                'Order Confirmed',
                'Your order has been processed'
            );
        }
    }
}
```

#### Python
```python
from typing import Type, TypeVar, Dict, Any
from abc import ABC, abstractmethod

T = TypeVar('T')

class ServiceLocator:
    _services: Dict[Type, Any] = {}
    
    @classmethod
    def register(cls, service_type: Type[T], implementation: T) -> None:
        cls._services[service_type] = implementation
    
    @classmethod
    def get_service(cls, service_type: Type[T]) -> T:
        service = cls._services.get(service_type)
        if not service:
            raise KeyError(f"Service not found: {service_type.__name__}")
        return service
    
    @classmethod
    def clear(cls) -> None:
        cls._services.clear()

# Service interfaces
class EmailService(ABC):
    @abstractmethod
    def send_email(self, to: str, subject: str, body: str) -> None:
        pass

class PaymentService(ABC):
    @abstractmethod
    def process_payment(self, order: 'Order') -> bool:
        pass

# Usage
class OrderProcessor:
    def __init__(self):
        self.email_service = ServiceLocator.get_service(EmailService)
        self.payment_service = ServiceLocator.get_service(PaymentService)
    
    def process(self, order: 'Order') -> None:
        if self.payment_service.process_payment(order):
            self.email_service.send_email(
                order.email,
                'Order Confirmed',
                'Your order has been processed'
            )
```

#### C#
```csharp
// Service interfaces
public interface IEmailService
{
    void SendEmail(string to, string subject, string body);
}

public interface IPaymentService
{
    Task<bool> ProcessPaymentAsync(Order order);
}

// Service Locator
public static class ServiceLocator
{
    private static readonly ConcurrentDictionary<Type, object> _services = 
        new ConcurrentDictionary<Type, object>();
    
    public static void Register<T>(T implementation)
        where T : class
    {
        _services.TryAdd(typeof(T), implementation);
    }
    
    public static T GetService<T>()
        where T : class
    {
        if (_services.TryGetValue(typeof(T), out var service))
        {
            return (T)service;
        }
        
        throw new InvalidOperationException($"Service not found: {typeof(T).Name}");
    }
    
    public static void Clear()
    {
        _services.Clear();
    }
}

// Usage
public class OrderProcessor
{
    private readonly IEmailService _emailService;
    private readonly IPaymentService _paymentService;
    
    public OrderProcessor()
    {
        _emailService = ServiceLocator.GetService<IEmailService>();
        _paymentService = ServiceLocator.GetService<IPaymentService>();
    }
    
    public async Task ProcessAsync(Order order)
    {
        if (await _paymentService.ProcessPaymentAsync(order))
        {
            _emailService.SendEmail(
                order.Email,
                "Order Confirmed",
                "Your order has been processed"
            );
        }
    }
}
```

**Key Differences:**
- Java uses type erasure and casting
- TypeScript leverages generics
- Python uses type hints and ABC
- C# has strong typing with generics

**WHEN SHOULD YOU USE IT?**
- When you need runtime service discovery
- When you can't use dependency injection
- When services are configured dynamically
- When testing with different implementations
- When you need a service registry

**WHERE YOU'VE ALREADY SEEN IT**
- JNDI in Java EE
- Service discovery in microservices
- Plugin architectures
- Legacy applications
- Testing frameworks

**WARNING**
While Service Locator is a valid pattern, many modern applications prefer Dependency Injection because:
- It makes dependencies explicit
- It's easier to test
- It's more maintainable
- It provides better compile-time safety

Consider Service Locator when:
- You can't modify existing code
- You need runtime flexibility
- You're building a plugin system
- You're working with legacy code
