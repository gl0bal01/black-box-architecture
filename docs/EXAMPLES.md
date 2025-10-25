# Code Examples

Real-world before/after examples showing black box refactoring in multiple languages.

See the `examples/` directory for complete, runnable examples.

## Quick Examples

### Python: Repository Pattern

**Before (Coupled to Database)**:
```python
import psycopg2

class UserService:
    def get_user(self, user_id):
        conn = psycopg2.connect("dbname=mydb")
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
        result = cursor.fetchone()
        conn.close()
        return result

    def save_user(self, user):
        conn = psycopg2.connect("dbname=mydb")
        cursor = conn.cursor()
        cursor.execute("INSERT INTO users ...")
        conn.commit()
        conn.close()
```

**After (Black Box Repository)**:
```python
from abc import ABC, abstractmethod

# Black box interface
class UserRepository(ABC):
    @abstractmethod
    def get_user(self, user_id: int) -> Optional[User]:
        pass

    @abstractmethod
    def save_user(self, user: User) -> None:
        pass

# Implementation hidden
class PostgresUserRepository(UserRepository):
    def __init__(self, connection_string: str):
        self.conn_string = connection_string

    def get_user(self, user_id: int) -> Optional[User]:
        # Implementation details
        pass

# Service uses interface only
class UserService:
    def __init__(self, repository: UserRepository):
        self.repo = repository

    def get_user(self, user_id: int) -> Optional[User]:
        return self.repo.get_user(user_id)
```

**Benefits**:
- ✅ Can swap database (MySQL, MongoDB, etc.)
- ✅ Easy to test (MockUserRepository)
- ✅ No SQL in business logic
- ✅ Database changes don't affect UserService

---

### TypeScript: Wrapping External APIs

**Before (Direct Dependency)**:
```typescript
import axios from 'axios';

class PaymentService {
  async processPayment(amount: number) {
    // Tightly coupled to axios
    const response = await axios.post('https://api.stripe.com/charges', {
      amount,
      currency: 'usd',
    });
    return response.data;
  }
}
```

**After (Black Box HTTP Client)**:
```typescript
// Your interface
interface HttpClient {
  post<T>(url: string, data: any): Promise<T>;
  get<T>(url: string): Promise<T>;
}

// Axios wrapper
class AxiosHttpClient implements HttpClient {
  async post<T>(url: string, data: any): Promise<T> {
    const response = await axios.post(url, data);
    return response.data;
  }
}

// Service uses interface
class PaymentService {
  constructor(private http: HttpClient) {}

  async processPayment(amount: number) {
    return this.http.post('https://api.stripe.com/charges', {
      amount,
      currency: 'usd',
    });
  }
}
```

**Benefits**:
- ✅ Can swap HTTP library (fetch, got, etc.)
- ✅ Easy to mock for testing
- ✅ Can add retry logic in wrapper
- ✅ Not tied to axios version

---

### Go: Interface-Driven Design

**Before (Concrete Implementation)**:
```go
type Logger struct {
    file *os.File
}

func (l *Logger) Log(message string) {
    l.file.WriteString(message + "\n")
}

// OrderService is stuck with file logging
type OrderService struct {
    logger *Logger
}
```

**After (Black Box Logger)**:
```go
// Interface first
type Logger interface {
    Info(message string)
    Error(message string)
}

// File implementation
type FileLogger struct {
    file *os.File
}

func (l *FileLogger) Info(message string) {
    l.file.WriteString("[INFO] " + message + "\n")
}

// Cloud implementation
type CloudLogger struct {
    client *cloudlogging.Client
}

func (l *CloudLogger) Info(message string) {
    l.client.Log(cloudlogging.Entry{Payload: message})
}

// Service uses interface
type OrderService struct {
    logger Logger
}

func NewOrderService(logger Logger) *OrderService {
    return &OrderService{logger: logger}
}
```

---

### Rust: Trait-Based Black Boxes

**Before (Hard-Coded Storage)**:
```rust
use std::fs;

pub struct ConfigManager {
    file_path: String,
}

impl ConfigManager {
    pub fn save(&self, config: &Config) -> Result<()> {
        let json = serde_json::to_string(config)?;
        fs::write(&self.file_path, json)?;
        Ok(())
    }
}
```

**After (Black Box Storage)**:
```rust
// Trait (interface)
pub trait Storage {
    fn write(&self, key: &str, data: &[u8]) -> Result<()>;
    fn read(&self, key: &str) -> Result<Vec<u8>>;
}

// File implementation
pub struct FileStorage {
    base_path: PathBuf,
}

impl Storage for FileStorage {
    fn write(&self, key: &str, data: &[u8]) -> Result<()> {
        fs::write(self.base_path.join(key), data)?;
        Ok(())
    }
}

// Config manager uses trait
pub struct ConfigManager<S: Storage> {
    storage: S,
}

impl<S: Storage> ConfigManager<S> {
    pub fn save(&self, config: &Config) -> Result<()> {
        let json = serde_json::to_string(config)?;
        self.storage.write("config.json", json.as_bytes())?;
        Ok(())
    }
}
```

---

For complete, runnable examples, see:
- [Python Examples](../examples/python/)
- [TypeScript Examples](../examples/typescript/)
- [Go Examples](../examples/go/)
- [Rust Examples](../examples/rust/)
