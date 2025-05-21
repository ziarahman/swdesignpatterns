# Design Patterns: Creational - Factory

## Factory Pattern - Simple Explanations

**SITUATION: YOU’RE CREATING OBJECTS**  
You’re building a feature that sends notifications. Initially, you have one type: `EmailNotification`. 

You create it like: 

#### Java
```java
new EmailNotification(to, subject, body);
``` 
All cool.

**THEN THE FEATURE GROWS**  
Now you have:  
- `SMSNotification`  
- `PushNotification`  
- `SlackNotification`  

Each type needs different parameters and setup logic. You’re stuck writing code like: 

#### Java
```java
if (type == "email") 
    new EmailNotification(to, subject, body); 
else if (type == "sms") 
    new SMSNotification(phone, message);
``` 

Everywhere.

**WHAT’S THE PROBLEM?**  
- This logic is repeated all over your codebase.  
- Every time you add a new type, you have to change existing code—breaking the Open/Closed Principle.  
- Code becomes tightly coupled to specific classes.  
- It’s a testing nightmare and hard to maintain.

**HOW FACTORY SAVES YOU**  
Instead of creating objects directly or writing switch/if chains, you create a `NotificationFactory`. You say: “Hey Factory, give me a notification object of type X.” And the Factory:  
- Hides the creation logic.  
- Decides which subclass to return.  
- Keeps your main code clean and decoupled.  
Now you don’t care how it creates it. You just ask and get a ready-to-use object: `Notification notification = NotificationFactory.create(type);`

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
interface Notification {  
    void send();  
}  
class EmailNotification implements Notification {  
    EmailNotification(String to, String subject, String body) {}  
    public void send() {  
        System.out.println("Sending email");  
    }  
}  
class SMSNotification implements Notification {  
    SMSNotification(String phone, String message) {}  
    public void send() {  
        System.out.println("Sending SMS");  
    }  
}  
class NotificationFactory {  
    public static Notification create(String type) {  
        if (type.equals("email")) return new EmailNotification("to", "subject", "body");  
        if (type.equals("sms")) return new SMSNotification("1234567890", "message");  
        throw new IllegalArgumentException("Unknown type");  
    }  
}  
// Usage  
Notification notification = NotificationFactory.create("email");  
notification.send();  
```

#### TypeScript  
```typescript  
interface Notification {  
    send(): void;  
}  
class EmailNotification implements Notification {  
    constructor(to: string, subject: string, body: string) {}  
    send() {  
        console.log("Sending email");  
    }  
}  
class SMSNotification implements Notification {  
    constructor(phone: string, message: string) {}  
    send() {  
        console.log("Sending SMS");  
    }  
}  
class NotificationFactory {  
    static create(type: string): Notification {  
        if (type === "email") return new EmailNotification("to", "subject", "body");  
        if (type === "sms") return new SMSNotification("1234567890", "message");  
        throw new Error("Unknown notification type");  
    }  
}  
// Usage  
const notification = NotificationFactory.create("email");  
notification.send();  
```

#### Python  
```python  
from abc import ABC, abstractmethod  
class Notification(ABC):  
    @abstractmethod  
    def send(self):  
        pass  
class EmailNotification(Notification):  
    def __init__(self, to, subject, body):  
        self.to, self.subject, self.body = to, subject, body  
    def send(self):  
        print("Sending email")  
class SMSNotification(Notification):  
    def __init__(self, phone, message):  
        self.phone, self.message = phone, message  
    def send(self):  
        print("Sending SMS")  
class NotificationFactory:  
    @staticmethod  
    def create(type):  
        if type == "email":  
            return EmailNotification("to", "subject", "body")  
        if type == "sms":  
            return SMSNotification("1234567890", "message")  
        raise ValueError("Unknown notification type")  
# Usage  
notification = NotificationFactory.create("email")  
notification.send()  
```

#### Go  
```go  
package main  
import "fmt"  
type Notification interface {  
    Send()  
}  
type EmailNotification struct {  
    to, subject, body string  
}  
func (e *EmailNotification) Send() {  
    fmt.Println("Sending email")  
}  
type SMSNotification struct {  
    phone, message string  
}  
func (s *SMSNotification) Send() {  
    fmt.Println("Sending SMS")  
}  
type NotificationFactory struct{}  
func (f *NotificationFactory) Create(type_ string) Notification {  
    if type_ == "email" {  
        return &EmailNotification{"to", "subject", "body"}  
    }  
    if type_ == "sms" {  
        return &SMSNotification{"1234567890", "message"}  
    }  
    panic("Unknown notification type")  
}  
// Usage  
func main() {  
    factory := &NotificationFactory{}  
    notification := factory.Create("email")  
    notification.Send()  
}  
```

**Key Differences:**  
- Java and TypeScript use interfaces and static methods for factory creation, emphasizing type safety.  
- Python leverages its dynamic nature, often using abstract base classes for structure.  
- Go uses interfaces but avoids inheritance, relying on composition and simple conditionals.  
- All achieve the same goal: decoupling object creation from usage.

**WHEN SHOULD YOU USE IT?**  
- When object creation is complex or involves conditional logic.  
- When you want to decouple creation from usage.  
- When you might add new types in the future, and don’t want to break existing logic.

**WHERE YOU’VE ALREADY SEEN IT**  
`Calendar.getInstance()` in Java.  
`LoggerFactory.getLogger()` in logging frameworks.  
`ConnectionFactory` in DB access layers.