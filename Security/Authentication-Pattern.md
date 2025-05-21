# Authentication Pattern

## Simple Explanation
Think of a security guard checking IDs at a building entrance. The Authentication pattern works similarly - it verifies the identity of users or systems before allowing access. Just as the guard might check multiple forms of ID (badge, photo ID, biometrics), the pattern supports various authentication methods.

## Real-World Scenario
Consider a web application's authentication:

Without Pattern:
```typescript
class UserController {
    async handleRequest(req: Request) {
        // Authentication mixed with business logic
        const token = req.headers.authorization;
        if (!token) {
            throw new Error('No token provided');
        }

        try {
            const decoded = jwt.verify(token, 'secret');
            const user = await this.userService.findById(decoded.userId);
            if (!user) {
                throw new Error('User not found');
            }

            // Finally, business logic
            return this.processUserRequest(user, req);
        } catch (error) {
            throw new Error('Authentication failed');
        }
    }
}
```

Problems:
- Mixed responsibilities
- Repeated authentication code
- Hard to change auth method
- Security vulnerabilities
- Complex testing

With Pattern:
```typescript
// Authentication logic isolated
@Injectable()
class AuthenticationService {
    async authenticate(credentials: Credentials): Promise<User> {
        const strategy = this.getStrategy(credentials.type);
        const user = await strategy.authenticate(credentials);
        return this.createSession(user);
    }
}

// Business logic stays clean
@Controller()
@UseGuards(AuthGuard)
class UserController {
    async handleRequest(req: AuthenticatedRequest) {
        // Business logic only
        const user = req.user;
        return this.processUserRequest(user, req);
    }
}
```

Benefits:
- Separated concerns
- Flexible auth methods
- Clear security boundary
- Easy to test
- Reusable components

## Problem Statement
Applications need to:
- Verify user identity
- Support multiple auth methods
- Maintain security state
- Handle auth failures
- Protect resources
But doing this leads to:
- Scattered auth code
- Security vulnerabilities
- Complex maintenance
- Poor testability

## Solution
The Authentication pattern provides:
1. Identity Verification
   - Multiple strategies
   - Secure protocols
   - Flexible methods

2. Session Management
   - Secure storage
   - Token handling
   - State management

3. Security Boundary
   - Clear separation
   - Consistent enforcement
   - Central configuration

## Implementation Examples

### TypeScript Implementation
```typescript
// Authentication Types
interface Credentials {
    type: 'password' | 'oauth' | 'jwt';
    [key: string]: any;
}

interface AuthenticationResult {
    user: User;
    token: string;
}

// Strategy Pattern for Different Auth Methods
interface AuthStrategy {
    authenticate(credentials: Credentials): Promise<User>;
}

class PasswordStrategy implements AuthStrategy {
    constructor(private userService: UserService) {}

    async authenticate(credentials: Credentials): Promise<User> {
        const { email, password } = credentials;
        const user = await this.userService.findByEmail(email);
        
        if (!user || !await this.verifyPassword(password, user.password)) {
            throw new AuthenticationError('Invalid credentials');
        }

        return user;
    }
}

// Main Authentication Service
class AuthenticationService {
    private strategies: Map<string, AuthStrategy>;
    private tokenService: TokenService;

    constructor() {
        this.strategies = new Map();
        this.registerStrategies();
    }

    async authenticate(
        credentials: Credentials
    ): Promise<AuthenticationResult> {
        const strategy = this.strategies.get(credentials.type);
        if (!strategy) {
            throw new AuthenticationError(
                `Unknown authentication type: ${credentials.type}`
            );
        }

        try {
            const user = await strategy.authenticate(credentials);
            const token = await this.tokenService.createToken(user);
            return { user, token };
        } catch (error) {
            throw new AuthenticationError(
                'Authentication failed', 
                { cause: error }
            );
        }
    }

    async verify(token: string): Promise<User> {
        return this.tokenService.verifyAndGetUser(token);
    }
}

// Authentication Middleware
class AuthenticationMiddleware {
    constructor(private authService: AuthenticationService) {}

    async handle(req: Request, res: Response, next: NextFunction) {
        const token = this.extractToken(req);
        if (!token) {
            throw new AuthenticationError('No token provided');
        }

        try {
            const user = await this.authService.verify(token);
            req.user = user;
            next();
        } catch (error) {
            throw new AuthenticationError('Invalid token');
        }
    }
}
```

### Java Implementation
```java
// Authentication Strategy
public interface AuthenticationStrategy {
    User authenticate(Credentials credentials) 
        throws AuthenticationException;
}

@Service
public class PasswordStrategy implements AuthenticationStrategy {
    private final UserService userService;
    private final PasswordEncoder passwordEncoder;

    public User authenticate(Credentials credentials) 
        throws AuthenticationException {
        String email = credentials.getEmail();
        String password = credentials.getPassword();

        User user = userService.findByEmail(email)
            .orElseThrow(() -> new AuthenticationException(
                "User not found"
            ));

        if (!passwordEncoder.matches(password, user.getPassword())) {
            throw new AuthenticationException("Invalid password");
        }

        return user;
    }
}

// Authentication Service
@Service
public class AuthenticationService {
    private final Map<String, AuthenticationStrategy> strategies;
    private final TokenService tokenService;

    public AuthenticationResult authenticate(Credentials credentials) 
        throws AuthenticationException {
        AuthenticationStrategy strategy = strategies.get(
            credentials.getType()
        );
        
        if (strategy == null) {
            throw new AuthenticationException(
                "Unknown authentication type"
            );
        }

        try {
            User user = strategy.authenticate(credentials);
            String token = tokenService.createToken(user);
            return new AuthenticationResult(user, token);
        } catch (Exception e) {
            throw new AuthenticationException(
                "Authentication failed", e
            );
        }
    }
}

// Security Configuration
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            .and()
            .addFilter(new JwtAuthenticationFilter(authService))
            .sessionManagement()
                .sessionCreationPolicy(STATELESS);
    }
}
```

### Python Implementation
```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Optional, Dict
from datetime import datetime, timedelta
import jwt

# Authentication Types
@dataclass
class Credentials:
    type: str
    email: Optional[str] = None
    password: Optional[str] = None
    token: Optional[str] = None

@dataclass
class AuthenticationResult:
    user: 'User'
    token: str

# Authentication Strategy
class AuthenticationStrategy(ABC):
    @abstractmethod
    async def authenticate(
        self, 
        credentials: Credentials
    ) -> 'User':
        pass

class PasswordStrategy(AuthenticationStrategy):
    def __init__(self, user_service: 'UserService'):
        self.user_service = user_service

    async def authenticate(
        self, 
        credentials: Credentials
    ) -> 'User':
        user = await self.user_service.find_by_email(
            credentials.email
        )
        if not user:
            raise AuthenticationError("User not found")

        if not await self.verify_password(
            credentials.password, 
            user.password
        ):
            raise AuthenticationError("Invalid password")

        return user

# Authentication Service
class AuthenticationService:
    def __init__(
        self,
        token_service: 'TokenService'
    ):
        self.strategies: Dict[str, AuthenticationStrategy] = {}
        self.token_service = token_service

    def register_strategy(
        self, 
        auth_type: str, 
        strategy: AuthenticationStrategy
    ):
        self.strategies[auth_type] = strategy

    async def authenticate(
        self, 
        credentials: Credentials
    ) -> AuthenticationResult:
        strategy = self.strategies.get(credentials.type)
        if not strategy:
            raise AuthenticationError(
                f"Unknown authentication type: {credentials.type}"
            )

        try:
            user = await strategy.authenticate(credentials)
            token = await self.token_service.create_token(user)
            return AuthenticationResult(user, token)
        except Exception as e:
            raise AuthenticationError(
                "Authentication failed"
            ) from e

    async def verify(self, token: str) -> 'User':
        return await self.token_service.verify_and_get_user(token)

# Authentication Middleware
class AuthenticationMiddleware:
    def __init__(self, auth_service: AuthenticationService):
        self.auth_service = auth_service

    async def __call__(
        self, 
        request: 'Request', 
        call_next: 'CallNext'
    ):
        token = self.extract_token(request)
        if not token:
            raise AuthenticationError("No token provided")

        try:
            user = await self.auth_service.verify(token)
            request.user = user
            return await call_next(request)
        except Exception as e:
            raise AuthenticationError("Invalid token") from e
```

## Usage Guidelines

1. **When to Use**
   - User authentication needed
   - Multiple auth methods
   - Resource protection
   - Security boundaries
   - Session management

2. **Best Practices**
   - Use proven libraries
   - Implement rate limiting
   - Secure password storage
   - Token-based auth
   - Regular security audits

3. **Implementation Considerations**
   - Session management
   - Token storage
   - Password hashing
   - Security headers
   - HTTPS enforcement

## Real-World Examples

1. **OAuth/OpenID Connect**
   - Social login
   - Token-based auth
   - Scope management
   - User consent
   - Identity federation

2. **JWT Authentication**
   - Stateless tokens
   - Claims-based auth
   - Digital signatures
   - Token expiration
   - Refresh tokens

3. **Multi-Factor Auth**
   - Password + OTP
   - Biometric verification
   - Hardware tokens
   - SMS verification
   - Email confirmation

## Key Implementation Differences

1. **TypeScript vs Java**
   - TypeScript: Middleware pattern
   - Java: Filter chain
   - Both support strategies

2. **Python vs Others**
   - Python: Async/await
   - Middleware as callables
   - Similar strategy pattern

3. **Security Features**
   - TypeScript: JWT focus
   - Java: Spring Security
   - Python: Framework agnostic
