# Design Patterns: Architectural - Repository

## Repository Pattern - Simple Explanation

**SITUATION: YOU'RE ACCESSING DATA**  
You have code that needs to work with data storage. 

Initially, you have: 

#### TypeScript
```typescript
class UserService {
    getUser(id: number) {
        return db.query('SELECT * FROM users WHERE id = ?', [id]);
    }
}
``` 
All good.

**THEN THE COMPLEXITY GROWS**  
Now you need:  
- Multiple data sources
- Different storage types
- Complex queries
- Caching
- Testing with mock data

You're stuck with: 
```typescript
class UserService {
    async getUser(id: number) {
        let user = await cache.get(`user:${id}`);
        if (!user) {
            user = await db.query('SELECT * FROM users WHERE id = ?', [id]);
            await cache.set(`user:${id}`, user);
        }
        return this.mapToModel(user);
    }
    
    private mapToModel(data: any) {
        // Complex mapping logic
    }
}
``` 
Data access code everywhere!

**WHAT'S THE PROBLEM?**  
- Data access logic scattered across the application
- Hard to switch data sources
- Duplicate query logic
- Complex testing
- No separation of concerns

**HOW REPOSITORY SAVES YOU**  
Create a dedicated layer for data access.

You say: "Repository, handle all data access details." 

And the Repository:  
- Abstracts data storage
- Centralizes data access logic
- Makes testing easier
- Separates concerns

Now you just: 
```typescript
class UserRepository {
    async getById(id: number): Promise<User> {
        return this.findOne({ id });
    }
}

// Business logic just uses the repository
const user = await userRepository.getById(123);
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### TypeScript  
```typescript  
// Entity
interface User {
    id: number;
    name: string;
    email: string;
}

// Repository interface
interface IUserRepository {
    getById(id: number): Promise<User>;
    findAll(): Promise<User[]>;
    create(user: User): Promise<User>;
    update(user: User): Promise<User>;
    delete(id: number): Promise<void>;
}

// Concrete repository implementation
class SqlUserRepository implements IUserRepository {
    constructor(private db: Database) {}
    
    async getById(id: number): Promise<User> {
        const result = await this.db.query(
            'SELECT * FROM users WHERE id = ?',
            [id]
        );
        return this.mapToEntity(result[0]);
    }
    
    async findAll(): Promise<User[]> {
        const results = await this.db.query('SELECT * FROM users');
        return results.map(this.mapToEntity);
    }
    
    async create(user: User): Promise<User> {
        const result = await this.db.query(
            'INSERT INTO users (name, email) VALUES (?, ?)',
            [user.name, user.email]
        );
        return { ...user, id: result.insertId };
    }
    
    private mapToEntity(row: any): User {
        return {
            id: row.id,
            name: row.name,
            email: row.email
        };
    }
}

// Usage in service
class UserService {
    constructor(private userRepository: IUserRepository) {}
    
    async createUser(userData: Omit<User, 'id'>): Promise<User> {
        const existingUser = await this.userRepository
            .findAll()
            .then(users => users.find(u => u.email === userData.email));
            
        if (existingUser) {
            throw new Error('Email already exists');
        }
        
        return this.userRepository.create(userData as User);
    }
}
```

#### C#  
```csharp  
// Entity
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

// Repository interface
public interface IUserRepository
{
    Task<User> GetByIdAsync(int id);
    Task<IEnumerable<User>> GetAllAsync();
    Task<User> CreateAsync(User user);
    Task<User> UpdateAsync(User user);
    Task DeleteAsync(int id);
}

// Concrete repository
public class EFUserRepository : IUserRepository
{
    private readonly AppDbContext _context;
    
    public EFUserRepository(AppDbContext context)
    {
        _context = context;
    }
    
    public async Task<User> GetByIdAsync(int id)
    {
        return await _context.Users
            .FirstOrDefaultAsync(u => u.Id == id);
    }
    
    public async Task<IEnumerable<User>> GetAllAsync()
    {
        return await _context.Users.ToListAsync();
    }
    
    public async Task<User> CreateAsync(User user)
    {
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        return user;
    }
    
    public async Task<User> UpdateAsync(User user)
    {
        _context.Entry(user).State = EntityState.Modified;
        await _context.SaveChangesAsync();
        return user;
    }
}

// Service using repository
public class UserService
{
    private readonly IUserRepository _userRepository;
    
    public UserService(IUserRepository userRepository)
    {
        _userRepository = userRepository;
    }
    
    public async Task<User> CreateUserAsync(User user)
    {
        var existingUser = await _userRepository.GetAllAsync()
            .FirstOrDefault(u => u.Email == user.Email);
            
        if (existingUser != null)
        {
            throw new InvalidOperationException("Email already exists");
        }
        
        return await _userRepository.CreateAsync(user);
    }
}
```

#### Python  
```python  
from abc import ABC, abstractmethod
from typing import List, Optional
from dataclasses import dataclass

@dataclass
class User:
    id: Optional[int]
    name: str
    email: str

class UserRepository(ABC):
    @abstractmethod
    def get_by_id(self, id: int) -> Optional[User]:
        pass
    
    @abstractmethod
    def get_all(self) -> List[User]:
        pass
    
    @abstractmethod
    def create(self, user: User) -> User:
        pass
    
    @abstractmethod
    def update(self, user: User) -> User:
        pass
    
    @abstractmethod
    def delete(self, id: int) -> None:
        pass

class SqlUserRepository(UserRepository):
    def __init__(self, db_connection):
        self.db = db_connection
    
    def get_by_id(self, id: int) -> Optional[User]:
        cursor = self.db.cursor()
        cursor.execute(
            "SELECT id, name, email FROM users WHERE id = ?",
            (id,)
        )
        row = cursor.fetchone()
        return self._map_to_entity(row) if row else None
    
    def create(self, user: User) -> User:
        cursor = self.db.cursor()
        cursor.execute(
            "INSERT INTO users (name, email) VALUES (?, ?)",
            (user.name, user.email)
        )
        self.db.commit()
        user.id = cursor.lastrowid
        return user
    
    def _map_to_entity(self, row) -> User:
        return User(
            id=row[0],
            name=row[1],
            email=row[2]
        )

# Service using repository
class UserService:
    def __init__(self, user_repository: UserRepository):
        self.user_repository = user_repository
    
    def create_user(self, user: User) -> User:
        existing_users = self.user_repository.get_all()
        if any(u.email == user.email for u in existing_users):
            raise ValueError("Email already exists")
        
        return self.user_repository.create(user)
```

#### Go  
```go  
package main

import (
    "context"
    "errors"
)

// Entity
type User struct {
    ID    int
    Name  string
    Email string
}

// Repository interface
type UserRepository interface {
    GetByID(ctx context.Context, id int) (*User, error)
    GetAll(ctx context.Context) ([]*User, error)
    Create(ctx context.Context, user *User) (*User, error)
    Update(ctx context.Context, user *User) (*User, error)
    Delete(ctx context.Context, id int) error
}

// Concrete repository
type SQLUserRepository struct {
    db *sql.DB
}

func NewSQLUserRepository(db *sql.DB) UserRepository {
    return &SQLUserRepository{db: db}
}

func (r *SQLUserRepository) GetByID(ctx context.Context, id int) (*User, error) {
    row := r.db.QueryRowContext(ctx,
        "SELECT id, name, email FROM users WHERE id = ?", id)
    
    user := &User{}
    err := row.Scan(&user.ID, &user.Name, &user.Email)
    if err == sql.ErrNoRows {
        return nil, nil
    }
    return user, err
}

func (r *SQLUserRepository) Create(ctx context.Context, user *User) (*User, error) {
    result, err := r.db.ExecContext(ctx,
        "INSERT INTO users (name, email) VALUES (?, ?)",
        user.Name, user.Email)
    if err != nil {
        return nil, err
    }
    
    id, err := result.LastInsertId()
    if err != nil {
        return nil, err
    }
    
    user.ID = int(id)
    return user, nil
}

// Service using repository
type UserService struct {
    repo UserRepository
}

func NewUserService(repo UserRepository) *UserService {
    return &UserService{repo: repo}
}

func (s *UserService) CreateUser(ctx context.Context, user *User) (*User, error) {
    existing, err := s.repo.GetAll(ctx)
    if err != nil {
        return nil, err
    }
    
    for _, u := range existing {
        if u.Email == user.Email {
            return nil, errors.New("email already exists")
        }
    }
    
    return s.repo.Create(ctx, user)
}
```

**Key Differences:**  
- TypeScript uses async/await with Promises
- C# leverages Entity Framework with LINQ
- Python uses abstract base classes
- Go implements through interfaces and context

**WHEN SHOULD YOU USE IT?**  
- When you need data access abstraction
- When working with multiple data sources
- When you want to centralize data logic
- When testing needs data mocking

**WHERE YOU'VE ALREADY SEEN IT**  
- ORM frameworks
- Data access layers
- Enterprise applications
- Microservice architectures
