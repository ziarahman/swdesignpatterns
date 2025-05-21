# Design Patterns: Enterprise - Data Transfer Object (DTO)

## DTO Pattern - Simple Explanation

**SITUATION: YOU'RE EXPOSING DATA**  
You have a rich domain model with lots of data and behavior.

Initially, you have:

#### TypeScript
```typescript
class Customer {
    id: number;
    firstName: string;
    lastName: string;
    email: string;
    password: string;
    creditCardNumber: string;
    shippingAddress: Address;
    billingAddress: Address;
    orders: Order[];
    
    calculateLoyaltyPoints(): number {
        // Complex calculation
        return points;
    }
}

@Controller('customers')
class CustomerController {
    @Get(':id')
    getCustomer(id: number): Customer {
        return this.customerService.findById(id);
    }
}
```

**THEN THE PROBLEMS START**  
- Sensitive data exposure
- Over-fetching of data
- Network bandwidth waste
- Tight coupling
- Security vulnerabilities

**HOW DTO SAVES YOU**  
Create specific data carriers for different use cases.

You say: "I'll only transfer the data I need in the format I need it."

And DTOs:
- Hide internal details
- Reduce data transfer
- Decouple layers
- Improve security
- Optimize for specific use cases

Now you have:
```typescript
class CustomerDTO {
    id: number;
    fullName: string;
    email: string;
    loyaltyPoints: number;
}

@Controller('customers')
class CustomerController {
    @Get(':id')
    getCustomer(id: number): CustomerDTO {
        const customer = this.customerService.findById(id);
        return {
            id: customer.id,
            fullName: `${customer.firstName} ${customer.lastName}`,
            email: customer.email,
            loyaltyPoints: customer.calculateLoyaltyPoints()
        };
    }
}
```

## Implementation Examples

#### TypeScript
```typescript
// Domain Model
class Customer {
    constructor(
        public id: number,
        public firstName: string,
        public lastName: string,
        public email: string,
        public password: string,
        private creditCard: CreditCard,
        public address: Address
    ) {}
    
    calculateLoyaltyPoints(): number {
        // Complex calculation
        return points;
    }
}

// DTOs
interface CustomerListDTO {
    id: number;
    fullName: string;
    email: string;
}

interface CustomerDetailsDTO {
    id: number;
    firstName: string;
    lastName: string;
    email: string;
    address: AddressDTO;
    loyaltyPoints: number;
}

// Mapper
class CustomerMapper {
    static toListDTO(customer: Customer): CustomerListDTO {
        return {
            id: customer.id,
            fullName: `${customer.firstName} ${customer.lastName}`,
            email: customer.email
        };
    }
    
    static toDetailsDTO(customer: Customer): CustomerDetailsDTO {
        return {
            id: customer.id,
            firstName: customer.firstName,
            lastName: customer.lastName,
            email: customer.email,
            address: AddressMapper.toDTO(customer.address),
            loyaltyPoints: customer.calculateLoyaltyPoints()
        };
    }
}
```

#### C#
```csharp
// Domain Model
public class Customer
{
    public int Id { get; private set; }
    public string FirstName { get; private set; }
    public string LastName { get; private set; }
    public string Email { get; private set; }
    public string Password { get; private set; }
    private CreditCard _creditCard;
    public Address Address { get; private set; }
    
    public decimal CalculateLoyaltyPoints() 
    {
        // Complex calculation
        return points;
    }
}

// DTOs
public record CustomerListDto(
    int Id,
    string FullName,
    string Email
);

public record CustomerDetailsDto(
    int Id,
    string FirstName,
    string LastName,
    string Email,
    AddressDto Address,
    decimal LoyaltyPoints
);

// Mapper
public static class CustomerMapper
{
    public static CustomerListDto ToListDto(Customer customer)
        => new(
            customer.Id,
            $"{customer.FirstName} {customer.LastName}",
            customer.Email
        );
    
    public static CustomerDetailsDto ToDetailsDto(Customer customer)
        => new(
            customer.Id,
            customer.FirstName,
            customer.LastName,
            customer.Email,
            AddressMapper.ToDto(customer.Address),
            customer.CalculateLoyaltyPoints()
        );
}
```

#### Java
```java
// Domain Model
public class Customer {
    private Long id;
    private String firstName;
    private String lastName;
    private String email;
    private String password;
    private CreditCard creditCard;
    private Address address;
    
    public BigDecimal calculateLoyaltyPoints() {
        // Complex calculation
        return points;
    }
    
    // Getters and setters
}

// DTOs
@Value
public class CustomerListDTO {
    Long id;
    String fullName;
    String email;
}

@Value
public class CustomerDetailsDTO {
    Long id;
    String firstName;
    String lastName;
    String email;
    AddressDTO address;
    BigDecimal loyaltyPoints;
}

// Mapper using MapStruct
@Mapper(componentModel = "spring")
public interface CustomerMapper {
    @Mapping(target = "fullName", 
             expression = "java(customer.getFirstName() + ' ' + customer.getLastName())")
    CustomerListDTO toListDTO(Customer customer);
    
    CustomerDetailsDTO toDetailsDTO(Customer customer);
}
```

#### Python
```python
from dataclasses import dataclass
from decimal import Decimal

# Domain Model
class Customer:
    def __init__(
        self,
        id: int,
        first_name: str,
        last_name: str,
        email: str,
        password: str,
        credit_card: 'CreditCard',
        address: 'Address'
    ):
        self.id = id
        self.first_name = first_name
        self.last_name = last_name
        self.email = email
        self._password = password
        self._credit_card = credit_card
        self.address = address
    
    def calculate_loyalty_points(self) -> Decimal:
        # Complex calculation
        return points

# DTOs
@dataclass
class CustomerListDTO:
    id: int
    full_name: str
    email: str

@dataclass
class CustomerDetailsDTO:
    id: int
    first_name: str
    last_name: str
    email: str
    address: 'AddressDTO'
    loyalty_points: Decimal

# Mapper
class CustomerMapper:
    @staticmethod
    def to_list_dto(customer: Customer) -> CustomerListDTO:
        return CustomerListDTO(
            id=customer.id,
            full_name=f"{customer.first_name} {customer.last_name}",
            email=customer.email
        )
    
    @staticmethod
    def to_details_dto(customer: Customer) -> CustomerDetailsDTO:
        return CustomerDetailsDTO(
            id=customer.id,
            first_name=customer.first_name,
            last_name=customer.last_name,
            email=customer.email,
            address=AddressMapper.to_dto(customer.address),
            loyalty_points=customer.calculate_loyalty_points()
        )
```

**Key Differences:**
- TypeScript uses interfaces for DTOs
- C# leverages records for immutable DTOs
- Java uses Lombok's @Value and MapStruct
- Python utilizes dataclasses

**WHEN SHOULD YOU USE IT?**
- When transferring data across boundaries
- When protecting sensitive information
- When optimizing network performance
- When decoupling layers
- When creating APIs

**WHERE YOU'VE ALREADY SEEN IT**
- REST APIs
- GraphQL resolvers
- Microservices communication
- Mobile app responses
- Reports generation
