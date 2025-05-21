# Authorization Pattern

## Simple Explanation
Think of a hotel key card system. After authenticating (proving you're a guest), your key card only opens specific doors - your room, the gym, but not other guests' rooms or staff areas. Similarly, the Authorization pattern controls what authenticated users can access or do within a system.

## Real-World Scenario
Consider a document management system:

Without Pattern:
```typescript
class DocumentService {
    async getDocument(user: User, docId: string) {
        const doc = await this.documentRepo.findById(docId);
        
        // Authorization logic scattered throughout
        if (doc.type === 'internal' && !user.isEmployee) {
            throw new Error('Access denied');
        }
        if (doc.status === 'confidential' && 
            !user.department === doc.department) {
            throw new Error('Access denied');
        }
        if (doc.type === 'financial' && 
            !user.roles.includes('finance')) {
            throw new Error('Access denied');
        }

        return doc;
    }

    async updateDocument(user: User, docId: string, changes: any) {
        // Same checks duplicated
        const doc = await this.documentRepo.findById(docId);
        if (doc.type === 'internal' && !user.isEmployee) {
            throw new Error('Access denied');
        }
        // More duplicated checks...
    }
}
```

Problems:
- Scattered authorization logic
- Duplicated rules
- Hard to maintain
- Inconsistent enforcement
- Complex testing

With Pattern:
```typescript
// Centralized authorization rules
@Injectable()
class DocumentPolicy {
    canView(user: User, document: Document): boolean {
        return this.satisfiesRules(user, document, 'view');
    }

    canEdit(user: User, document: Document): boolean {
        return this.satisfiesRules(user, document, 'edit');
    }

    private satisfiesRules(
        user: User, 
        document: Document, 
        action: string
    ): boolean {
        const rules = this.getRulesForAction(action);
        return rules.every(rule => rule.evaluate(user, document));
    }
}

// Clean business logic
@Controller()
class DocumentController {
    constructor(private policy: DocumentPolicy) {}

    @Get(':id')
    async getDocument(req: AuthRequest, docId: string) {
        const doc = await this.documentRepo.findById(docId);
        if (!this.policy.canView(req.user, doc)) {
            throw new ForbiddenError();
        }
        return doc;
    }
}
```

Benefits:
- Centralized rules
- Consistent enforcement
- Easy to maintain
- Clear permissions
- Testable policies

## Problem Statement
Applications need to:
- Control resource access
- Enforce business rules
- Handle multiple roles
- Manage permissions
- Protect operations
But doing this leads to:
- Scattered rules
- Inconsistent enforcement
- Complex maintenance
- Difficult testing

## Solution
The Authorization pattern provides:
1. Access Control
   - Role-based access
   - Permission management
   - Resource protection
   - Operation control

2. Policy Management
   - Centralized rules
   - Clear permissions
   - Consistent enforcement
   - Easy updates

3. Security Boundary
   - Clear access limits
   - Explicit denials
   - Audit capability

## Implementation Examples

### TypeScript Implementation
```typescript
// Core Types
interface User {
    id: string;
    roles: string[];
    department: string;
    permissions: Set<string>;
}

interface Resource {
    id: string;
    type: string;
    ownerId: string;
    department: string;
    status: string;
}

// Policy Interface
interface IAuthorizationPolicy<T extends Resource> {
    authorize(user: User, resource: T, action: string): boolean;
}

// Rule-based Policy Implementation
class AuthorizationPolicy<T extends Resource> 
implements IAuthorizationPolicy<T> {
    private rules: Map<string, AuthorizationRule<T>[]>;

    constructor() {
        this.rules = new Map();
        this.setupRules();
    }

    authorize(user: User, resource: T, action: string): boolean {
        const rules = this.rules.get(action) || [];
        return rules.every(rule => rule.evaluate(user, resource));
    }

    private setupRules() {
        // View rules
        this.rules.set('view', [
            new DepartmentRule(),
            new RoleBasedRule(),
            new ResourceStatusRule()
        ]);

        // Edit rules
        this.rules.set('edit', [
            new DepartmentRule(),
            new RoleBasedRule(),
            new ResourceStatusRule(),
            new OwnershipRule()
        ]);
    }
}

// Individual Rules
abstract class AuthorizationRule<T extends Resource> {
    abstract evaluate(user: User, resource: T): boolean;
}

class DepartmentRule<T extends Resource> 
extends AuthorizationRule<T> {
    evaluate(user: User, resource: T): boolean {
        return user.department === resource.department;
    }
}

class RoleBasedRule<T extends Resource> 
extends AuthorizationRule<T> {
    evaluate(user: User, resource: T): boolean {
        switch (resource.type) {
            case 'financial':
                return user.roles.includes('finance');
            case 'hr':
                return user.roles.includes('hr');
            default:
                return true;
        }
    }
}

// Authorization Service
@Injectable()
class AuthorizationService {
    private policies: Map<string, IAuthorizationPolicy<any>>;

    constructor() {
        this.policies = new Map();
        this.registerPolicies();
    }

    authorize<T extends Resource>(
        user: User,
        resource: T,
        action: string
    ): boolean {
        const policy = this.policies.get(resource.type);
        if (!policy) {
            throw new Error(`No policy for ${resource.type}`);
        }

        return policy.authorize(user, resource, action);
    }

    private registerPolicies() {
        this.policies.set('document', new DocumentPolicy());
        this.policies.set('project', new ProjectPolicy());
    }
}
```

### Java Implementation
```java
// Core Interfaces
public interface AuthorizationPolicy<T extends Resource> {
    boolean authorize(User user, T resource, String action);
}

// Rule-based Implementation
@Service
public class DocumentAuthorizationPolicy 
implements AuthorizationPolicy<Document> {
    private final Map<String, List<AuthorizationRule<Document>>> rules;

    public DocumentAuthorizationPolicy() {
        this.rules = new HashMap<>();
        setupRules();
    }

    @Override
    public boolean authorize(
        User user, 
        Document document, 
        String action
    ) {
        List<AuthorizationRule<Document>> actionRules = 
            rules.getOrDefault(action, Collections.emptyList());

        return actionRules.stream()
            .allMatch(rule -> rule.evaluate(user, document));
    }

    private void setupRules() {
        rules.put("view", Arrays.asList(
            new DepartmentRule(),
            new RoleBasedRule(),
            new DocumentStatusRule()
        ));

        rules.put("edit", Arrays.asList(
            new DepartmentRule(),
            new RoleBasedRule(),
            new DocumentStatusRule(),
            new OwnershipRule()
        ));
    }
}

// Authorization Service
@Service
public class AuthorizationService {
    private final Map<Class<?>, AuthorizationPolicy<?>> policies;

    public <T extends Resource> boolean authorize(
        User user, 
        T resource, 
        String action
    ) {
        @SuppressWarnings("unchecked")
        AuthorizationPolicy<T> policy = 
            (AuthorizationPolicy<T>) policies.get(
                resource.getClass()
            );

        if (policy == null) {
            throw new AuthorizationException(
                "No policy found for " + resource.getClass()
            );
        }

        return policy.authorize(user, resource, action);
    }
}

// Authorization Aspect
@Aspect
@Component
public class AuthorizationAspect {
    private final AuthorizationService authService;

    @Around("@annotation(requiresAuthorization)")
    public Object authorize(
        ProceedingJoinPoint joinPoint,
        RequiresAuthorization auth
    ) throws Throwable {
        User user = SecurityContextHolder.getContext()
            .getAuthentication()
            .getPrincipal();

        Resource resource = extractResource(joinPoint);
        
        if (!authService.authorize(user, resource, auth.action())) {
            throw new AccessDeniedException("Access denied");
        }

        return joinPoint.proceed();
    }
}
```

### Python Implementation
```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import List, Dict, Set, Type
from functools import wraps

# Core Types
@dataclass
class User:
    id: str
    roles: List[str]
    department: str
    permissions: Set[str]

@dataclass
class Resource:
    id: str
    type: str
    owner_id: str
    department: str
    status: str

# Authorization Rules
class AuthorizationRule(ABC):
    @abstractmethod
    def evaluate(
        self, 
        user: User, 
        resource: Resource
    ) -> bool:
        pass

class DepartmentRule(AuthorizationRule):
    def evaluate(
        self, 
        user: User, 
        resource: Resource
    ) -> bool:
        return user.department == resource.department

class RoleBasedRule(AuthorizationRule):
    def evaluate(
        self, 
        user: User, 
        resource: Resource
    ) -> bool:
        if resource.type == 'financial':
            return 'finance' in user.roles
        return True

# Policy Implementation
class AuthorizationPolicy:
    def __init__(self):
        self.rules: Dict[str, List[AuthorizationRule]] = {}
        self.setup_rules()

    def setup_rules(self):
        self.rules['view'] = [
            DepartmentRule(),
            RoleBasedRule()
        ]
        self.rules['edit'] = [
            DepartmentRule(),
            RoleBasedRule(),
            OwnershipRule()
        ]

    def authorize(
        self, 
        user: User, 
        resource: Resource, 
        action: str
    ) -> bool:
        rules = self.rules.get(action, [])
        return all(
            rule.evaluate(user, resource) 
            for rule in rules
        )

# Authorization Service
class AuthorizationService:
    def __init__(self):
        self.policies: Dict[str, AuthorizationPolicy] = {}

    def register_policy(
        self, 
        resource_type: str, 
        policy: AuthorizationPolicy
    ):
        self.policies[resource_type] = policy

    def authorize(
        self, 
        user: User, 
        resource: Resource, 
        action: str
    ) -> bool:
        policy = self.policies.get(resource.type)
        if not policy:
            raise ValueError(
                f"No policy for {resource.type}"
            )
        return policy.authorize(user, resource, action)

# Decorator for Authorization
def requires_authorization(action: str):
    def decorator(f):
        @wraps(f)
        async def wrapper(self, *args, **kwargs):
            user = await get_current_user()
            resource = await self.get_resource(**kwargs)
            
            if not await auth_service.authorize(
                user, resource, action
            ):
                raise ForbiddenError()
                
            return await f(self, *args, **kwargs)
        return wrapper
    return decorator
```

## Usage Guidelines

1. **When to Use**
   - Resource access control
   - Role-based permissions
   - Complex authorization rules
   - Multi-tenant systems
   - Regulatory compliance

2. **Best Practices**
   - Centralize policies
   - Default to deny
   - Audit access attempts
   - Clear role hierarchies
   - Regular policy reviews

3. **Implementation Considerations**
   - Rule granularity
   - Performance impact
   - Caching strategy
   - Role hierarchy
   - Policy updates

## Real-World Examples

1. **Document Management**
   - Role-based access
   - Department permissions
   - Document classification
   - Version control
   - Audit trails

2. **Healthcare Systems**
   - Patient record access
   - Role-based permissions
   - Department restrictions
   - Emergency access
   - Audit requirements

3. **Financial Systems**
   - Transaction limits
   - Approval workflows
   - Role hierarchies
   - Compliance rules
   - Audit logging

## Key Implementation Differences

1. **TypeScript vs Java**
   - TypeScript: Decorator-based
   - Java: Aspect-oriented
   - Both support type-safe rules

2. **Python vs Others**
   - Python: Decorator pattern
   - Async support
   - Similar policy structure

3. **Framework Integration**
   - TypeScript: NestJS guards
   - Java: Spring Security
   - Python: FastAPI dependencies
