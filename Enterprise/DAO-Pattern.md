# Design Patterns: Enterprise - DAO (Data Access Object)

## DAO Pattern - Simple Explanation

**SITUATION: YOU'RE ACCESSING A DATA STORE**  
You have code that needs to work with a database. 

Initially, you have: 

#### Java
```java
class UserService {
    void saveUser(User user) {
        String sql = "INSERT INTO users (name, email) VALUES (?, ?)";
        jdbc.execute(sql, user.getName(), user.getEmail());
    }
}
``` 
All good.

**THEN THE COMPLEXITY GROWS**  
Now you need:  
- Multiple data stores
- Complex queries
- Different data formats
- Connection management
- Error handling

You're stuck with: 
```java
class UserService {
    void saveUser(User user) {
        Connection conn = null;
        try {
            conn = dataSource.getConnection();
            PreparedStatement ps = conn.prepareStatement(
                "INSERT INTO users (name, email) VALUES (?, ?)"
            );
            ps.setString(1, user.getName());
            ps.setString(2, user.getEmail());
            ps.executeUpdate();
            ps.close();
        } catch (SQLException e) {
            // Handle error
        } finally {
            if (conn != null) {
                try {
                    conn.close();
                } catch (SQLException e) {
                    // Handle error
                }
            }
        }
    }
}
``` 
Database code everywhere!

**WHAT'S THE PROBLEM?**  
- Database access code scattered
- Hard to change data store
- Complex connection handling
- No separation of concerns
- Duplicated SQL code

**HOW DAO SAVES YOU**  
Create objects dedicated to data access.

You say: "DAO, handle all the database details." 

And the DAO:  
- Encapsulates data access
- Manages connections
- Handles specific database logic
- Makes testing easier
- Separates concerns

Now you just: 
```java
userDao.save(user);
```

**IMPLEMENTATION IN OTHER LANGUAGES**

#### Java  
```java  
// DAO Interface
public interface UserDao {
    User findById(Long id);
    List<User> findAll();
    void save(User user);
    void update(User user);
    void delete(Long id);
}

// DAO Implementation
public class UserDaoImpl implements UserDao {
    private final DataSource dataSource;
    
    public UserDaoImpl(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    
    @Override
    public User findById(Long id) {
        String sql = "SELECT * FROM users WHERE id = ?";
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {
            ps.setLong(1, id);
            try (ResultSet rs = ps.executeQuery()) {
                if (rs.next()) {
                    return mapRow(rs);
                }
            }
        } catch (SQLException e) {
            throw new DataAccessException("Error finding user", e);
        }
        return null;
    }
    
    @Override
    public void save(User user) {
        String sql = "INSERT INTO users (name, email) VALUES (?, ?)";
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {
            ps.setString(1, user.getName());
            ps.setString(2, user.getEmail());
            ps.executeUpdate();
        } catch (SQLException e) {
            throw new DataAccessException("Error saving user", e);
        }
    }
    
    private User mapRow(ResultSet rs) throws SQLException {
        User user = new User();
        user.setId(rs.getLong("id"));
        user.setName(rs.getString("name"));
        user.setEmail(rs.getString("email"));
        return user;
    }
}
```

#### TypeScript  
```typescript  
interface UserDao {
    findById(id: number): Promise<User>;
    findAll(): Promise<User[]>;
    save(user: User): Promise<void>;
    update(user: User): Promise<void>;
    delete(id: number): Promise<void>;
}

class PostgresUserDao implements UserDao {
    constructor(private pool: Pool) {}
    
    async findById(id: number): Promise<User> {
        const query = 'SELECT * FROM users WHERE id = $1';
        const result = await this.pool.query(query, [id]);
        return this.mapRow(result.rows[0]);
    }
    
    async save(user: User): Promise<void> {
        const query = `
            INSERT INTO users (name, email) 
            VALUES ($1, $2)
        `;
        await this.pool.query(query, [user.name, user.email]);
    }
    
    private mapRow(row: any): User {
        return {
            id: row.id,
            name: row.name,
            email: row.email
        };
    }
}

// Usage in service
class UserService {
    constructor(private userDao: UserDao) {}
    
    async createUser(user: User): Promise<void> {
        await this.userDao.save(user);
    }
}
```

#### Python  
```python  
from abc import ABC, abstractmethod
from typing import List, Optional
import mysql.connector
from contextlib import contextmanager

class UserDao(ABC):
    @abstractmethod
    def find_by_id(self, id: int) -> Optional['User']:
        pass
    
    @abstractmethod
    def find_all(self) -> List['User']:
        pass
    
    @abstractmethod
    def save(self, user: 'User') -> None:
        pass
    
    @abstractmethod
    def update(self, user: 'User') -> None:
        pass
    
    @abstractmethod
    def delete(self, id: int) -> None:
        pass

class MySQLUserDao(UserDao):
    def __init__(self, db_config: dict):
        self.db_config = db_config
    
    @contextmanager
    def get_connection(self):
        conn = mysql.connector.connect(**self.db_config)
        try:
            yield conn
        finally:
            conn.close()
    
    def find_by_id(self, id: int) -> Optional['User']:
        with self.get_connection() as conn:
            cursor = conn.cursor()
            cursor.execute(
                "SELECT * FROM users WHERE id = %s", 
                (id,)
            )
            row = cursor.fetchone()
            return self._map_row(row) if row else None
    
    def save(self, user: 'User') -> None:
        with self.get_connection() as conn:
            cursor = conn.cursor()
            cursor.execute(
                "INSERT INTO users (name, email) VALUES (%s, %s)",
                (user.name, user.email)
            )
            conn.commit()
    
    def _map_row(self, row: tuple) -> 'User':
        return User(
            id=row[0],
            name=row[1],
            email=row[2]
        )

# Usage
class UserService:
    def __init__(self, user_dao: UserDao):
        self.user_dao = user_dao
    
    def create_user(self, user: User):
        self.user_dao.save(user)
```

#### Go  
```go  
package main

import (
    "context"
    "database/sql"
)

// UserDAO interface
type UserDAO interface {
    FindByID(ctx context.Context, id int64) (*User, error)
    FindAll(ctx context.Context) ([]*User, error)
    Save(ctx context.Context, user *User) error
    Update(ctx context.Context, user *User) error
    Delete(ctx context.Context, id int64) error
}

// SQLUserDAO implements UserDAO
type SQLUserDAO struct {
    db *sql.DB
}

func NewSQLUserDAO(db *sql.DB) UserDAO {
    return &SQLUserDAO{db: db}
}

func (dao *SQLUserDAO) FindByID(ctx context.Context, id int64) (*User, error) {
    row := dao.db.QueryRowContext(ctx,
        "SELECT id, name, email FROM users WHERE id = $1",
        id,
    )
    
    user := &User{}
    err := row.Scan(&user.ID, &user.Name, &user.Email)
    if err == sql.ErrNoRows {
        return nil, nil
    }
    if err != nil {
        return nil, err
    }
    return user, nil
}

func (dao *SQLUserDAO) Save(ctx context.Context, user *User) error {
    _, err := dao.db.ExecContext(ctx,
        "INSERT INTO users (name, email) VALUES ($1, $2)",
        user.Name, user.Email,
    )
    return err
}

// UserService uses DAO
type UserService struct {
    userDAO UserDAO
}

func (s *UserService) CreateUser(ctx context.Context, user *User) error {
    return s.userDAO.Save(ctx, user)
}
```

**Key Differences:**  
- Java uses JDBC with try-with-resources
- TypeScript implements with async/await
- Python uses context managers
- Go leverages sql.DB and context

**WHEN SHOULD YOU USE IT?**  
- When abstracting data storage details
- When working with multiple data sources
- When you need clean separation of concerns
- When data access needs to be standardized

**WHERE YOU'VE ALREADY SEEN IT**  
- ORM frameworks
- Spring JDBC template
- Enterprise applications
- Microservices data layers
