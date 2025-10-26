# Go Black Box Examples

Complete before/after examples demonstrating black box architecture transformations in Go.

## Example 1: User Service

### Before (Tightly Coupled)

```go
package main

import (
    "database/sql"
    "errors"
    "net/smtp"
    _ "github.com/lib/pq"
    "golang.org/x/crypto/bcrypt"
)

type UserManager struct {
    // Hard-coded dependencies
    db *sql.DB
    smtpHost string
    smtpAuth smtp.Auth
}

func NewUserManager() (*UserManager, error) {
    // Direct database connection
    db, err := sql.Open("postgres", "host=localhost dbname=myapp user=postgres password=secret")
    if err != nil {
        return nil, err
    }

    // Hard-coded SMTP
    auth := smtp.PlainAuth("", "noreply@example.com", "password", "smtp.gmail.com")

    return &UserManager{
        db:       db,
        smtpHost: "smtp.gmail.com:587",
        smtpAuth: auth,
    }, nil
}

func (um *UserManager) RegisterUser(email, password, name string) (int, error) {
    // Validation mixed with everything
    if len(email) == 0 || len(password) < 8 {
        return 0, errors.New("invalid input")
    }

    // Direct database query
    var exists bool
    err := um.db.QueryRow("SELECT EXISTS(SELECT 1 FROM users WHERE email = $1)", email).Scan(&exists)
    if err != nil {
        return 0, err
    }
    if exists {
        return 0, errors.New("email exists")
    }

    // Password hashing
    hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    if err != nil {
        return 0, err
    }

    // Insert user
    var userID int
    err = um.db.QueryRow(
        "INSERT INTO users (email, password, name) VALUES ($1, $2, $3) RETURNING id",
        email, string(hash), name,
    ).Scan(&userID)
    if err != nil {
        return 0, err
    }

    // Send email - hard-coded SMTP
    msg := []byte("Subject: Welcome\r\n\r\nThanks for signing up!")
    err = smtp.SendMail(um.smtpHost, um.smtpAuth, "noreply@example.com", []string{email}, msg)
    if err != nil {
        return 0, err
    }

    return userID, nil
}
```

**Problems:**
- ❌ Can't test without PostgreSQL
- ❌ Can't test without SMTP
- ❌ Can't swap database
- ❌ Can't swap email provider
- ❌ Everything coupled together
- ❌ Changes cascade

### After (Black Box Architecture)

```go
package main

import (
    "context"
    "errors"
)

// ===== PRIMITIVES (Core Data Types) =====

type User struct {
    ID           int
    Email        string
    PasswordHash string
    Name         string
}

// ===== BLACK BOX INTERFACES =====

// UserRepository defines WHAT operations are needed, not HOW
type UserRepository interface {
    FindByEmail(ctx context.Context, email string) (*User, error)
    Save(ctx context.Context, user *User) (int, error)
}

// EmailService defines email operations
type EmailService interface {
    SendWelcome(ctx context.Context, email, name string) error
}

// PasswordHasher defines password operations
type PasswordHasher interface {
    Hash(password string) (string, error)
    Verify(password, hash string) error
}

// ===== IMPLEMENTATIONS (Replaceable) =====

// PostgresUserRepository implements UserRepository
type PostgresUserRepository struct {
    db *sql.DB
}

func NewPostgresUserRepository(db *sql.DB) *PostgresUserRepository {
    return &PostgresUserRepository{db: db}
}

func (r *PostgresUserRepository) FindByEmail(ctx context.Context, email string) (*User, error) {
    var user User
    err := r.db.QueryRowContext(ctx,
        "SELECT id, email, password, name FROM users WHERE email = $1",
        email,
    ).Scan(&user.ID, &user.Email, &user.PasswordHash, &user.Name)

    if err == sql.ErrNoRows {
        return nil, nil // Not found
    }
    if err != nil {
        return nil, err
    }

    return &user, nil
}

func (r *PostgresUserRepository) Save(ctx context.Context, user *User) (int, error) {
    var id int
    err := r.db.QueryRowContext(ctx,
        "INSERT INTO users (email, password, name) VALUES ($1, $2, $3) RETURNING id",
        user.Email, user.PasswordHash, user.Name,
    ).Scan(&id)

    return id, err
}

// BcryptHasher implements PasswordHasher
type BcryptHasher struct{}

func NewBcryptHasher() *BcryptHasher {
    return &BcryptHasher{}
}

func (h *BcryptHasher) Hash(password string) (string, error) {
    hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    return string(hash), err
}

func (h *BcryptHasher) Verify(password, hash string) error {
    return bcrypt.CompareHashAndPassword([]byte(hash), []byte(password))
}

// SMTPEmailService implements EmailService
type SMTPEmailService struct {
    host string
    auth smtp.Auth
}

func NewSMTPEmailService(host string, auth smtp.Auth) *SMTPEmailService {
    return &SMTPEmailService{host: host, auth: auth}
}

func (s *SMTPEmailService) SendWelcome(ctx context.Context, email, name string) error {
    msg := []byte("Subject: Welcome " + name + "\r\n\r\nThanks for signing up!")
    return smtp.SendMail(s.host, s.auth, "noreply@example.com", []string{email}, msg)
}

// ===== BUSINESS LOGIC (Single Responsibility) =====

// UserRegistrationService handles user registration
type UserRegistrationService struct {
    userRepo     UserRepository
    emailService EmailService
    hasher       PasswordHasher
}

func NewUserRegistrationService(
    userRepo UserRepository,
    emailService EmailService,
    hasher PasswordHasher,
) *UserRegistrationService {
    return &UserRegistrationService{
        userRepo:     userRepo,
        emailService: emailService,
        hasher:       hasher,
    }
}

func (s *UserRegistrationService) Register(ctx context.Context, email, password, name string) (int, error) {
    // Validate
    if len(email) == 0 || len(password) < 8 {
        return 0, errors.New("invalid input")
    }

    // Check if exists
    existing, err := s.userRepo.FindByEmail(ctx, email)
    if err != nil {
        return 0, err
    }
    if existing != nil {
        return 0, errors.New("email exists")
    }

    // Hash password
    hash, err := s.hasher.Hash(password)
    if err != nil {
        return 0, err
    }

    // Create user
    user := &User{
        Email:        email,
        PasswordHash: hash,
        Name:         name,
    }

    // Save
    userID, err := s.userRepo.Save(ctx, user)
    if err != nil {
        return 0, err
    }

    // Send welcome email
    if err := s.emailService.SendWelcome(ctx, email, name); err != nil {
        // Log error but don't fail registration
        // In production, use a proper logger
    }

    return userID, nil
}
```

**Benefits:**
- ✅ **Testable**: Mock all interfaces
- ✅ **Replaceable**: Swap PostgreSQL → MySQL, SMTP → SendGrid
- ✅ **Clear**: Each interface has one job
- ✅ **Idiomatic Go**: Interfaces at usage point
- ✅ **Context-aware**: Proper cancellation/timeout support

### Testing the Black Box Version

```go
package main

import (
    "context"
    "testing"
)

// ===== MOCK IMPLEMENTATIONS =====

type InMemoryUserRepository struct {
    users  map[string]*User
    nextID int
}

func NewInMemoryUserRepository() *InMemoryUserRepository {
    return &InMemoryUserRepository{
        users:  make(map[string]*User),
        nextID: 1,
    }
}

func (r *InMemoryUserRepository) FindByEmail(ctx context.Context, email string) (*User, error) {
    user, ok := r.users[email]
    if !ok {
        return nil, nil
    }
    return user, nil
}

func (r *InMemoryUserRepository) Save(ctx context.Context, user *User) (int, error) {
    user.ID = r.nextID
    r.nextID++
    r.users[user.Email] = user
    return user.ID, nil
}

type MockEmailService struct {
    sentEmails []string
}

func NewMockEmailService() *MockEmailService {
    return &MockEmailService{
        sentEmails: make([]string, 0),
    }
}

func (m *MockEmailService) SendWelcome(ctx context.Context, email, name string) error {
    m.sentEmails = append(m.sentEmails, email)
    return nil
}

type FakeHasher struct{}

func NewFakeHasher() *FakeHasher {
    return &FakeHasher{}
}

func (h *FakeHasher) Hash(password string) (string, error) {
    return "hashed_" + password, nil
}

func (h *FakeHasher) Verify(password, hash string) error {
    if hash == "hashed_"+password {
        return nil
    }
    return errors.New("invalid password")
}

// ===== TESTS =====

func TestRegister_Success(t *testing.T) {
    // Arrange
    userRepo := NewInMemoryUserRepository()
    emailService := NewMockEmailService()
    hasher := NewFakeHasher()

    service := NewUserRegistrationService(userRepo, emailService, hasher)

    // Act
    userID, err := service.Register(context.Background(), "test@example.com", "password123", "John")

    // Assert
    if err != nil {
        t.Fatalf("expected no error, got %v", err)
    }

    if userID != 1 {
        t.Errorf("expected userID 1, got %d", userID)
    }

    if len(emailService.sentEmails) != 1 {
        t.Errorf("expected 1 email sent, got %d", len(emailService.sentEmails))
    }

    // Verify user was saved
    savedUser, _ := userRepo.FindByEmail(context.Background(), "test@example.com")
    if savedUser == nil {
        t.Fatal("user not saved")
    }

    if savedUser.Name != "John" {
        t.Errorf("expected name John, got %s", savedUser.Name)
    }
}

func TestRegister_DuplicateEmail(t *testing.T) {
    service := NewUserRegistrationService(
        NewInMemoryUserRepository(),
        NewMockEmailService(),
        NewFakeHasher(),
    )

    // Register first user
    _, err := service.Register(context.Background(), "test@example.com", "password123", "John")
    if err != nil {
        t.Fatal(err)
    }

    // Try duplicate
    _, err = service.Register(context.Background(), "test@example.com", "password456", "Jane")
    if err == nil {
        t.Fatal("expected error for duplicate email")
    }
}
```

---

## Example 2: HTTP Client with Retry

### Before (Tightly Coupled)

```go
import (
    "encoding/json"
    "net/http"
    "time"
)

type APIClient struct {
    httpClient *http.Client
    baseURL    string
}

func NewAPIClient() *APIClient {
    return &APIClient{
        httpClient: &http.Client{Timeout: 10 * time.Second},
        baseURL:    "https://api.example.com",
    }
}

func (c *APIClient) GetProduct(id string) (*Product, error) {
    // Retry logic mixed with HTTP logic
    var lastErr error
    for i := 0; i < 3; i++ {
        resp, err := c.httpClient.Get(c.baseURL + "/products/" + id)
        if err != nil {
            lastErr = err
            time.Sleep(time.Second)
            continue
        }
        defer resp.Body.Close()

        if resp.StatusCode != 200 {
            lastErr = errors.New("non-200 status")
            time.Sleep(time.Second)
            continue
        }

        var product Product
        if err := json.NewDecoder(resp.Body).Decode(&product); err != nil {
            return nil, err
        }

        return &product, nil
    }

    return nil, lastErr
}
```

**Problems:**
- ❌ Can't test without real HTTP server
- ❌ Can't swap HTTP client
- ❌ Retry logic hard-coded
- ❌ No way to configure backoff

### After (Black Box Architecture)

```go
// ===== PRIMITIVES =====

type Product struct {
    ID    string  `json:"id"`
    Name  string  `json:"name"`
    Price float64 `json:"price"`
}

// ===== BLACK BOX INTERFACES =====

type HTTPClient interface {
    Get(ctx context.Context, url string) (*http.Response, error)
}

type RetryStrategy interface {
    Execute(ctx context.Context, fn func() error) error
}

type ProductRepository interface {
    GetByID(ctx context.Context, id string) (*Product, error)
}

// ===== IMPLEMENTATIONS =====

type DefaultHTTPClient struct {
    client *http.Client
}

func NewDefaultHTTPClient(timeout time.Duration) *DefaultHTTPClient {
    return &DefaultHTTPClient{
        client: &http.Client{Timeout: timeout},
    }
}

func (c *DefaultHTTPClient) Get(ctx context.Context, url string) (*http.Response, error) {
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, err
    }
    return c.client.Do(req)
}

type ExponentialBackoff struct {
    maxRetries int
    baseDelay  time.Duration
}

func NewExponentialBackoff(maxRetries int, baseDelay time.Duration) *ExponentialBackoff {
    return &ExponentialBackoff{
        maxRetries: maxRetries,
        baseDelay:  baseDelay,
    }
}

func (r *ExponentialBackoff) Execute(ctx context.Context, fn func() error) error {
    var lastErr error

    for i := 0; i < r.maxRetries; i++ {
        if err := fn(); err != nil {
            lastErr = err
            if i < r.maxRetries-1 {
                delay := r.baseDelay * time.Duration(1<<uint(i))
                select {
                case <-time.After(delay):
                case <-ctx.Done():
                    return ctx.Err()
                }
            }
            continue
        }
        return nil
    }

    return lastErr
}

type APIProductRepository struct {
    httpClient HTTPClient
    retry      RetryStrategy
    baseURL    string
}

func NewAPIProductRepository(
    httpClient HTTPClient,
    retry RetryStrategy,
    baseURL string,
) *APIProductRepository {
    return &APIProductRepository{
        httpClient: httpClient,
        retry:      retry,
        baseURL:    baseURL,
    }
}

func (r *APIProductRepository) GetByID(ctx context.Context, id string) (*Product, error) {
    var product *Product

    err := r.retry.Execute(ctx, func() error {
        resp, err := r.httpClient.Get(ctx, r.baseURL+"/products/"+id)
        if err != nil {
            return err
        }
        defer resp.Body.Close()

        if resp.StatusCode != 200 {
            return errors.New("non-200 status")
        }

        var p Product
        if err := json.NewDecoder(resp.Body).Decode(&p); err != nil {
            return err
        }

        product = &p
        return nil
    })

    return product, err
}
```

**Benefits:**
- ✅ **Testable**: Mock HTTP client and retry
- ✅ **Replaceable**: Swap HTTP clients, retry strategies
- ✅ **Flexible**: Change backoff algorithm
- ✅ **Context-aware**: Respects cancellation

### Testing Example

```go
type MockHTTPClient struct {
    responses []*http.Response
    errors    []error
    callCount int
}

func (m *MockHTTPClient) Get(ctx context.Context, url string) (*http.Response, error) {
    if m.callCount >= len(m.responses) {
        return nil, m.errors[m.callCount]
    }

    resp := m.responses[m.callCount]
    err := m.errors[m.callCount]
    m.callCount++
    return resp, err
}

type NoRetry struct{}

func (r *NoRetry) Execute(ctx context.Context, fn func() error) error {
    return fn()
}

func TestGetProduct(t *testing.T) {
    // Mock successful response
    body := `{"id": "1", "name": "Widget", "price": 9.99}`
    mock := &MockHTTPClient{
        responses: []*http.Response{
            {StatusCode: 200, Body: io.NopCloser(strings.NewReader(body))},
        },
        errors: []error{nil},
    }

    repo := NewAPIProductRepository(mock, &NoRetry{}, "https://api.example.com")

    product, err := repo.GetByID(context.Background(), "1")

    if err != nil {
        t.Fatal(err)
    }

    if product.Name != "Widget" {
        t.Errorf("expected Widget, got %s", product.Name)
    }
}
```

---

## Key Patterns in Go

### 1. Small Interfaces

```go
// Good - focused interface
type Reader interface {
    Read(p []byte) (n int, err error)
}

// Good - composition
type ReadWriter interface {
    Reader
    Writer
}

// Bad - too many methods
type DataStore interface {
    Create() error
    Read() error
    Update() error
    Delete() error
    List() error
    Search() error
    // ... many more
}
```

### 2. Accept Interfaces, Return Structs

```go
// Good
func ProcessData(reader io.Reader) (*Result, error) {
    // Accept interface - flexible
    // Return struct - concrete
    return &Result{}, nil
}

// Bad
func ProcessData(file *os.File) (io.Reader, error) {
    // Too specific input
    // Too generic output
    return nil, nil
}
```

### 3. Interface at Usage Point

```go
// Define interface where you use it, not where you implement it

// In package "service"
type UserRepository interface {
    FindByID(ctx context.Context, id int) (*User, error)
}

type UserService struct {
    repo UserRepository // Uses interface
}

// In package "postgres"
type PostgresUserRepository struct {
    db *sql.DB
}

// Implements service.UserRepository implicitly
func (r *PostgresUserRepository) FindByID(ctx context.Context, id int) (*User, error) {
    // ...
}
```

---

## Common Anti-Patterns to Avoid

### ❌ Interface Pollution

```go
// Bad - interface for everything
type UserRepositoryInterface interface {
    FindByID(id int) (*User, error)
}

type PostgresUserRepository struct{}

func (r *PostgresUserRepository) FindByID(id int) (*User, error) {
    // Only one implementation - interface not needed!
}
```

### ❌ God Struct

```go
// Bad - doing too much
type Application struct {
    db       *sql.DB
    redis    *redis.Client
    logger   *log.Logger
    mailer   *smtp.Client
    s3       *s3.Client
    // ... 20 more fields
}
```

### ❌ Concrete Dependencies

```go
// Bad - hard-coded dependencies
type OrderService struct {
    postgres *PostgresRepository // ❌ Concrete type
    stripe   *StripePayment      // ❌ Concrete type
}

// Good - interfaces
type OrderService struct {
    repo    OrderRepository // ✅ Interface
    payment PaymentGateway  // ✅ Interface
}
```

---

## Contributing

Add more Go examples via pull request!
