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

### C: Opaque Types (Eskil's Approach)

**Before (Exposed Structure)**:
```c
// user.h - Exposes implementation
typedef struct {
    char name[256];
    char email[256];
    int age;
} User;

void user_create(User* user, const char* name);
```

**After (Opaque Type)**:
```c
// user.h - Black box interface
typedef struct User User;  // Forward declaration only

// Public API
User* user_create(const char* name, const char* email);
void user_destroy(User* user);
const char* user_get_name(User* user);
void user_set_age(User* user, int age);
```

```c
// user.c - Implementation hidden
struct User {
    char* name;
    char* email;
    int age;
};

User* user_create(const char* name, const char* email) {
    User* user = malloc(sizeof(User));
    user->name = strdup(name);
    user->email = strdup(email);
    user->age = 0;
    return user;
}

void user_destroy(User* user) {
    free(user->name);
    free(user->email);
    free(user);
}
```

**Benefits**:
- ✅ Can change internal structure without breaking code
- ✅ Prevents direct field access
- ✅ True encapsulation in C

---

### PHP: Service Layer Pattern

**Before (Fat Controller)**:
```php
<?php
class UserController {
    public function register() {
        // Validation, business logic, DB, email all mixed
        if (!filter_var($_POST['email'], FILTER_VALIDATE_EMAIL)) {
            throw new Exception('Invalid email');
        }

        $pdo = new PDO('mysql:host=localhost;dbname=mydb', 'user', 'pass');
        $stmt = $pdo->prepare('INSERT INTO users (email, password) VALUES (?, ?)');
        $stmt->execute([$_POST['email'], password_hash($_POST['password'], PASSWORD_BCRYPT)]);

        mail($_POST['email'], 'Welcome', 'Thanks for registering!');
    }
}
```

**After (Black Box Service Layer)**:
```php
<?php
// Service encapsulates business logic
class UserRegistrationService {
    private UserRepositoryInterface $userRepo;
    private EmailServiceInterface $emailService;

    public function __construct(
        UserRepositoryInterface $userRepo,
        EmailServiceInterface $emailService
    ) {
        $this->userRepo = $userRepo;
        $this->emailService = $emailService;
    }

    public function register(array $data): array {
        // Validate
        if (!filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
            return ['success' => false, 'errors' => ['email' => 'Invalid email']];
        }

        // Save user
        $userId = $this->userRepo->save($data);

        // Send welcome email
        $this->emailService->send($data['email'], 'Welcome!', 'Thanks!');

        return ['success' => true, 'user_id' => $userId];
    }
}

// Thin controller
class UserController {
    private UserRegistrationService $registrationService;

    public function __construct(UserRegistrationService $service) {
        $this->registrationService = $service;
    }

    public function register() {
        $result = $this->registrationService->register($_POST);
        echo json_encode($result);
    }
}
```

**Benefits**:
- ✅ Easy to test with mocks
- ✅ Reusable business logic
- ✅ Thin controllers
- ✅ Swappable dependencies

---

For complete, runnable examples, see:
- [Python Examples](../examples/python/) - Repository pattern, service abstractions
- [TypeScript Examples](../examples/typescript/) - Interface-driven design, DI
- [Go Examples](../examples/go/) - Interface composition, struct patterns
- [Rust Examples](../examples/rust/) - Trait-based black boxes
- [C Examples](../examples/c/) - Opaque types, function pointers (Eskil's approach!)
- [PHP Examples](../examples/php/) - Service layer, strategy pattern, Laravel integration
