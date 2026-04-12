# Rust Black Box Examples

Complete before/after examples demonstrating black box architecture transformations in Rust using traits, generics, and dependency injection.

## Example 1: User Service

### Before (Tightly Coupled)

```rust
use postgres::{Client, NoTls};
use bcrypt::{hash, DEFAULT_COST};
use lettre::{Message, SmtpTransport, Transport};

pub struct UserManager {
    db: Client,
    smtp_host: String,
    smtp_user: String,
    smtp_pass: String,
}

impl UserManager {
    pub fn new() -> Result<Self, Box<dyn std::error::Error>> {
        // Hard-coded database connection
        let db = Client::connect(
            "host=localhost user=postgres password=secret dbname=myapp",
            NoTls,
        )?;

        Ok(UserManager {
            db,
            smtp_host: "smtp.gmail.com".to_string(),
            smtp_user: "noreply@example.com".to_string(),
            smtp_pass: "password".to_string(),
        })
    }

    pub fn register_user(
        &mut self,
        email: &str,
        password: &str,
        name: &str,
    ) -> Result<i32, Box<dyn std::error::Error>> {
        // Validation mixed with logic
        if email.is_empty() || password.len() < 8 {
            return Err("invalid input".into());
        }

        // Direct database query
        let exists: bool = self
            .db
            .query_one(
                "SELECT EXISTS(SELECT 1 FROM users WHERE email = $1)",
                &[&email],
            )?
            .get(0);

        if exists {
            return Err("email exists".into());
        }

        // Hard-coded password hashing
        let password_hash = hash(password, DEFAULT_COST)?;

        // Insert user
        let row = self.db.query_one(
            "INSERT INTO users (email, password, name) VALUES ($1, $2, $3) RETURNING id",
            &[&email, &password_hash, &name],
        )?;
        let user_id: i32 = row.get(0);

        // Hard-coded SMTP email
        let email_msg = Message::builder()
            .from(self.smtp_user.parse()?)
            .to(email.parse()?)
            .subject("Welcome")
            .body(format!("Thanks for signing up, {}!", name))?;

        let mailer = SmtpTransport::relay(&self.smtp_host)?
            .credentials((self.smtp_user.clone(), self.smtp_pass.clone()).into())
            .build();

        mailer.send(&email_msg)?;

        Ok(user_id)
    }
}
```

**Problems:**
- ❌ Can't test without PostgreSQL
- ❌ Can't test without SMTP server
- ❌ Can't swap database backend
- ❌ Can't swap email provider
- ❌ Validation, hashing, persistence, and notification all coupled
- ❌ Error types leak implementation details

### After (Black Box Architecture)

```rust
use async_trait::async_trait;
use thiserror::Error;

// ===== PRIMITIVES (Core Data Types) =====

#[derive(Debug, Clone)]
pub struct User {
    pub id: i32,
    pub email: String,
    pub password_hash: String,
    pub name: String,
}

#[derive(Debug, Error)]
pub enum RegistrationError {
    #[error("invalid input: {0}")]
    InvalidInput(String),
    #[error("email already exists")]
    EmailExists,
    #[error("storage error: {0}")]
    Storage(String),
    #[error("hashing error: {0}")]
    Hashing(String),
}

// ===== BLACK BOX TRAITS =====

// UserRepository defines WHAT, not HOW
#[async_trait]
pub trait UserRepository: Send + Sync {
    async fn find_by_email(&self, email: &str) -> Result<Option<User>, String>;
    async fn save(&self, user: &User) -> Result<i32, String>;
}

// EmailService abstracts notification
#[async_trait]
pub trait EmailService: Send + Sync {
    async fn send_welcome(&self, email: &str, name: &str) -> Result<(), String>;
}

// PasswordHasher abstracts credential handling
pub trait PasswordHasher: Send + Sync {
    fn hash(&self, password: &str) -> Result<String, String>;
    fn verify(&self, password: &str, hash: &str) -> Result<bool, String>;
}

// ===== IMPLEMENTATIONS (Replaceable) =====

pub struct PostgresUserRepository {
    pool: sqlx::PgPool,
}

impl PostgresUserRepository {
    pub fn new(pool: sqlx::PgPool) -> Self {
        Self { pool }
    }
}

#[async_trait]
impl UserRepository for PostgresUserRepository {
    async fn find_by_email(&self, email: &str) -> Result<Option<User>, String> {
        sqlx::query_as!(
            User,
            "SELECT id, email, password_hash, name FROM users WHERE email = $1",
            email
        )
        .fetch_optional(&self.pool)
        .await
        .map_err(|e| e.to_string())
    }

    async fn save(&self, user: &User) -> Result<i32, String> {
        let row = sqlx::query!(
            "INSERT INTO users (email, password_hash, name) VALUES ($1, $2, $3) RETURNING id",
            user.email,
            user.password_hash,
            user.name
        )
        .fetch_one(&self.pool)
        .await
        .map_err(|e| e.to_string())?;

        Ok(row.id)
    }
}

pub struct BcryptHasher {
    cost: u32,
}

impl BcryptHasher {
    pub fn new(cost: u32) -> Self {
        Self { cost }
    }
}

impl PasswordHasher for BcryptHasher {
    fn hash(&self, password: &str) -> Result<String, String> {
        bcrypt::hash(password, self.cost).map_err(|e| e.to_string())
    }

    fn verify(&self, password: &str, hash: &str) -> Result<bool, String> {
        bcrypt::verify(password, hash).map_err(|e| e.to_string())
    }
}

pub struct SmtpEmailService {
    mailer: lettre::SmtpTransport,
    from: String,
}

impl SmtpEmailService {
    pub fn new(mailer: lettre::SmtpTransport, from: String) -> Self {
        Self { mailer, from }
    }
}

#[async_trait]
impl EmailService for SmtpEmailService {
    async fn send_welcome(&self, email: &str, name: &str) -> Result<(), String> {
        let msg = lettre::Message::builder()
            .from(self.from.parse().map_err(|e: lettre::address::AddressError| e.to_string())?)
            .to(email.parse().map_err(|e: lettre::address::AddressError| e.to_string())?)
            .subject("Welcome")
            .body(format!("Thanks for signing up, {}!", name))
            .map_err(|e| e.to_string())?;

        lettre::Transport::send(&self.mailer, &msg)
            .map(|_| ())
            .map_err(|e| e.to_string())
    }
}

// ===== BUSINESS LOGIC (Single Responsibility) =====

pub struct UserRegistrationService<R, E, H>
where
    R: UserRepository,
    E: EmailService,
    H: PasswordHasher,
{
    repo: R,
    email: E,
    hasher: H,
}

impl<R, E, H> UserRegistrationService<R, E, H>
where
    R: UserRepository,
    E: EmailService,
    H: PasswordHasher,
{
    pub fn new(repo: R, email: E, hasher: H) -> Self {
        Self { repo, email, hasher }
    }

    pub async fn register(
        &self,
        email: &str,
        password: &str,
        name: &str,
    ) -> Result<i32, RegistrationError> {
        // Validate
        if email.is_empty() || password.len() < 8 {
            return Err(RegistrationError::InvalidInput(
                "email required, password >= 8 chars".into(),
            ));
        }

        // Check existence
        let existing = self
            .repo
            .find_by_email(email)
            .await
            .map_err(RegistrationError::Storage)?;

        if existing.is_some() {
            return Err(RegistrationError::EmailExists);
        }

        // Hash password
        let password_hash = self
            .hasher
            .hash(password)
            .map_err(RegistrationError::Hashing)?;

        // Save
        let user = User {
            id: 0,
            email: email.to_string(),
            password_hash,
            name: name.to_string(),
        };

        let user_id = self
            .repo
            .save(&user)
            .await
            .map_err(RegistrationError::Storage)?;

        // Send welcome (best-effort, don't fail registration)
        let _ = self.email.send_welcome(email, name).await;

        Ok(user_id)
    }
}
```

**Benefits:**
- ✅ **Testable**: Mock all traits with zero I/O
- ✅ **Replaceable**: Swap Postgres → MySQL, SMTP → SendGrid at the edge
- ✅ **Type-safe**: Generic bounds enforce trait contracts at compile time
- ✅ **Zero-cost**: Static dispatch when using generics (no vtable overhead)
- ✅ **Error boundaries**: Domain errors never leak driver-specific details

### Testing the Black Box Version

```rust
use std::sync::Mutex;
use std::collections::HashMap;

// ===== IN-MEMORY TEST DOUBLES =====

pub struct InMemoryUserRepository {
    users: Mutex<HashMap<String, User>>,
    next_id: Mutex<i32>,
}

impl InMemoryUserRepository {
    pub fn new() -> Self {
        Self {
            users: Mutex::new(HashMap::new()),
            next_id: Mutex::new(1),
        }
    }
}

#[async_trait]
impl UserRepository for InMemoryUserRepository {
    async fn find_by_email(&self, email: &str) -> Result<Option<User>, String> {
        Ok(self.users.lock().unwrap().get(email).cloned())
    }

    async fn save(&self, user: &User) -> Result<i32, String> {
        let mut id_lock = self.next_id.lock().unwrap();
        let id = *id_lock;
        *id_lock += 1;

        let mut stored = user.clone();
        stored.id = id;
        self.users.lock().unwrap().insert(user.email.clone(), stored);
        Ok(id)
    }
}

pub struct MockEmailService {
    sent: Mutex<Vec<String>>,
}

impl MockEmailService {
    pub fn new() -> Self {
        Self { sent: Mutex::new(vec![]) }
    }

    pub fn sent_count(&self) -> usize {
        self.sent.lock().unwrap().len()
    }
}

#[async_trait]
impl EmailService for MockEmailService {
    async fn send_welcome(&self, email: &str, _name: &str) -> Result<(), String> {
        self.sent.lock().unwrap().push(email.to_string());
        Ok(())
    }
}

pub struct FakeHasher;

impl PasswordHasher for FakeHasher {
    fn hash(&self, password: &str) -> Result<String, String> {
        Ok(format!("hashed_{}", password))
    }

    fn verify(&self, password: &str, hash: &str) -> Result<bool, String> {
        Ok(hash == format!("hashed_{}", password))
    }
}

// ===== TESTS =====

#[cfg(test)]
mod tests {
    use super::*;

    #[tokio::test]
    async fn test_register_success() {
        let repo = InMemoryUserRepository::new();
        let email = MockEmailService::new();
        let hasher = FakeHasher;

        let service = UserRegistrationService::new(repo, email, hasher);

        let user_id = service
            .register("test@example.com", "password123", "John")
            .await
            .expect("registration should succeed");

        assert_eq!(user_id, 1);
    }

    #[tokio::test]
    async fn test_register_duplicate_email() {
        let service = UserRegistrationService::new(
            InMemoryUserRepository::new(),
            MockEmailService::new(),
            FakeHasher,
        );

        service
            .register("test@example.com", "password123", "John")
            .await
            .unwrap();

        let result = service
            .register("test@example.com", "password456", "Jane")
            .await;

        assert!(matches!(result, Err(RegistrationError::EmailExists)));
    }

    #[tokio::test]
    async fn test_register_short_password() {
        let service = UserRegistrationService::new(
            InMemoryUserRepository::new(),
            MockEmailService::new(),
            FakeHasher,
        );

        let result = service.register("test@example.com", "short", "John").await;

        assert!(matches!(result, Err(RegistrationError::InvalidInput(_))));
    }
}
```

---

## Example 2: HTTP Client with Retry

### Before (Tightly Coupled)

```rust
use reqwest::blocking::Client;
use std::thread::sleep;
use std::time::Duration;

pub struct ApiClient {
    http: Client,
    base_url: String,
}

impl ApiClient {
    pub fn new() -> Self {
        Self {
            http: Client::builder()
                .timeout(Duration::from_secs(10))
                .build()
                .unwrap(),
            base_url: "https://api.example.com".to_string(),
        }
    }

    pub fn get_product(&self, id: &str) -> Result<Product, Box<dyn std::error::Error>> {
        // Retry logic tangled with HTTP logic
        let mut last_err = None;

        for _ in 0..3 {
            let url = format!("{}/products/{}", self.base_url, id);
            match self.http.get(&url).send() {
                Ok(resp) if resp.status().is_success() => {
                    return Ok(resp.json()?);
                }
                Ok(_) => {
                    last_err = Some("non-200 status".to_string());
                    sleep(Duration::from_secs(1));
                }
                Err(e) => {
                    last_err = Some(e.to_string());
                    sleep(Duration::from_secs(1));
                }
            }
        }

        Err(last_err.unwrap().into())
    }
}
```

**Problems:**
- ❌ Can't test without real HTTP server
- ❌ Retry strategy hard-coded (count, delay, no backoff)
- ❌ No way to swap transport
- ❌ Can't change retry policy per endpoint

### After (Black Box Architecture)

```rust
use async_trait::async_trait;
use serde::Deserialize;
use std::future::Future;
use std::pin::Pin;
use std::time::Duration;

// ===== PRIMITIVES =====

#[derive(Debug, Deserialize, Clone)]
pub struct Product {
    pub id: String,
    pub name: String,
    pub price: f64,
}

#[derive(Debug)]
pub struct HttpResponse {
    pub status: u16,
    pub body: Vec<u8>,
}

// ===== BLACK BOX TRAITS =====

#[async_trait]
pub trait HttpClient: Send + Sync {
    async fn get(&self, url: &str) -> Result<HttpResponse, String>;
}

#[async_trait]
pub trait RetryStrategy: Send + Sync {
    async fn execute<'a, F>(&self, op: F) -> Result<HttpResponse, String>
    where
        F: Fn() -> Pin<Box<dyn Future<Output = Result<HttpResponse, String>> + Send + 'a>>
            + Send
            + Sync
            + 'a;
}

#[async_trait]
pub trait ProductRepository: Send + Sync {
    async fn get_by_id(&self, id: &str) -> Result<Product, String>;
}

// ===== IMPLEMENTATIONS =====

pub struct ReqwestClient {
    client: reqwest::Client,
}

impl ReqwestClient {
    pub fn new(timeout: Duration) -> Self {
        Self {
            client: reqwest::Client::builder().timeout(timeout).build().unwrap(),
        }
    }
}

#[async_trait]
impl HttpClient for ReqwestClient {
    async fn get(&self, url: &str) -> Result<HttpResponse, String> {
        let resp = self.client.get(url).send().await.map_err(|e| e.to_string())?;
        let status = resp.status().as_u16();
        let body = resp.bytes().await.map_err(|e| e.to_string())?.to_vec();
        Ok(HttpResponse { status, body })
    }
}

pub struct ExponentialBackoff {
    max_retries: u32,
    base_delay: Duration,
}

impl ExponentialBackoff {
    pub fn new(max_retries: u32, base_delay: Duration) -> Self {
        Self { max_retries, base_delay }
    }
}

#[async_trait]
impl RetryStrategy for ExponentialBackoff {
    async fn execute<'a, F>(&self, op: F) -> Result<HttpResponse, String>
    where
        F: Fn() -> Pin<Box<dyn Future<Output = Result<HttpResponse, String>> + Send + 'a>>
            + Send
            + Sync
            + 'a,
    {
        let mut last_err = String::from("no attempts");

        for attempt in 0..self.max_retries {
            match op().await {
                Ok(resp) if resp.status < 500 => return Ok(resp),
                Ok(resp) => last_err = format!("status {}", resp.status),
                Err(e) => last_err = e,
            }

            if attempt < self.max_retries - 1 {
                let delay = self.base_delay * 2u32.pow(attempt);
                tokio::time::sleep(delay).await;
            }
        }

        Err(last_err)
    }
}

pub struct ApiProductRepository<C, R>
where
    C: HttpClient,
    R: RetryStrategy,
{
    http: C,
    retry: R,
    base_url: String,
}

impl<C, R> ApiProductRepository<C, R>
where
    C: HttpClient,
    R: RetryStrategy,
{
    pub fn new(http: C, retry: R, base_url: String) -> Self {
        Self { http, retry, base_url }
    }
}

#[async_trait]
impl<C, R> ProductRepository for ApiProductRepository<C, R>
where
    C: HttpClient + 'static,
    R: RetryStrategy + 'static,
{
    async fn get_by_id(&self, id: &str) -> Result<Product, String> {
        let url = format!("{}/products/{}", self.base_url, id);
        let http = &self.http;

        let resp = self
            .retry
            .execute(|| Box::pin(async { http.get(&url).await }))
            .await?;

        if resp.status != 200 {
            return Err(format!("unexpected status: {}", resp.status));
        }

        serde_json::from_slice::<Product>(&resp.body).map_err(|e| e.to_string())
    }
}
```

**Benefits:**
- ✅ **Testable**: Mock HTTP and retry independently
- ✅ **Flexible retry**: Swap exponential → linear → fixed at runtime
- ✅ **Transport-agnostic**: Switch reqwest → hyper → ureq without touching business logic
- ✅ **Observable**: Wrap traits with middleware (logging, metrics) transparently

### Testing Example

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::sync::Mutex;

    pub struct MockHttpClient {
        responses: Mutex<Vec<Result<HttpResponse, String>>>,
    }

    impl MockHttpClient {
        pub fn new(responses: Vec<Result<HttpResponse, String>>) -> Self {
            Self { responses: Mutex::new(responses) }
        }
    }

    #[async_trait]
    impl HttpClient for MockHttpClient {
        async fn get(&self, _url: &str) -> Result<HttpResponse, String> {
            self.responses
                .lock()
                .unwrap()
                .pop()
                .unwrap_or(Err("no more responses".into()))
        }
    }

    pub struct NoRetry;

    #[async_trait]
    impl RetryStrategy for NoRetry {
        async fn execute<'a, F>(&self, op: F) -> Result<HttpResponse, String>
        where
            F: Fn() -> Pin<Box<dyn Future<Output = Result<HttpResponse, String>> + Send + 'a>>
                + Send
                + Sync
                + 'a,
        {
            op().await
        }
    }

    #[tokio::test]
    async fn test_get_product_success() {
        let body = br#"{"id":"1","name":"Widget","price":9.99}"#.to_vec();
        let mock = MockHttpClient::new(vec![Ok(HttpResponse { status: 200, body })]);

        let repo = ApiProductRepository::new(mock, NoRetry, "https://api.example.com".into());

        let product = repo.get_by_id("1").await.expect("should succeed");
        assert_eq!(product.name, "Widget");
        assert_eq!(product.price, 9.99);
    }

    #[tokio::test]
    async fn test_get_product_not_found() {
        let mock = MockHttpClient::new(vec![Ok(HttpResponse {
            status: 404,
            body: vec![],
        })]);

        let repo = ApiProductRepository::new(mock, NoRetry, "https://api.example.com".into());

        let result = repo.get_by_id("missing").await;
        assert!(result.is_err());
    }
}
```

---

## Key Patterns in Rust

### 1. Traits as Black-Box Interfaces

```rust
// Good - focused trait, one responsibility
pub trait Storage {
    fn write(&self, key: &str, data: &[u8]) -> Result<(), StorageError>;
    fn read(&self, key: &str) -> Result<Vec<u8>, StorageError>;
}

// Good - compose smaller traits
pub trait ReadStorage {
    fn read(&self, key: &str) -> Result<Vec<u8>, StorageError>;
}

pub trait WriteStorage {
    fn write(&self, key: &str, data: &[u8]) -> Result<(), StorageError>;
}

pub trait ReadWriteStorage: ReadStorage + WriteStorage {}
```

### 2. Static vs Dynamic Dispatch

```rust
// Static dispatch - zero cost, monomorphized per type
pub struct ConfigManager<S: Storage> {
    storage: S,
}

// Dynamic dispatch - runtime polymorphism, heterogeneous collections
pub struct ConfigManager {
    storage: Box<dyn Storage>,
}

// Use static by default; switch to dyn when you need runtime selection
```

### 3. Constructor Injection

```rust
impl<S: Storage> ConfigManager<S> {
    pub fn new(storage: S) -> Self {
        // Dependencies come in through the constructor
        // Never reach out to globals or singletons
        Self { storage }
    }
}
```

### 4. Domain Errors at the Boundary

```rust
#[derive(Debug, thiserror::Error)]
pub enum ConfigError {
    #[error("not found: {0}")]
    NotFound(String),
    #[error("corrupt data")]
    Corrupt,
}

// Convert from driver errors at the edge - never leak them
impl From<sqlx::Error> for ConfigError {
    fn from(_: sqlx::Error) -> Self {
        ConfigError::Corrupt
    }
}
```

---

## Common Anti-Patterns to Avoid

### ❌ Trait Pollution

```rust
// Bad - trait for a type with only one implementation
pub trait UserRepositoryTrait {
    fn find(&self, id: i32) -> Result<User, Error>;
}

pub struct PostgresUserRepository;

impl UserRepositoryTrait for PostgresUserRepository {
    fn find(&self, id: i32) -> Result<User, Error> { todo!() }
}

// Good - only introduce a trait when you need substitution
// (testing, multiple backends, middleware)
```

### ❌ Concrete Dependencies

```rust
// Bad - hard-coded concrete types
pub struct OrderService {
    db: sqlx::PgPool,       // ❌ locked to Postgres
    stripe: stripe::Client, // ❌ locked to Stripe
}

// Good - depend on traits
pub struct OrderService<R: OrderRepository, P: PaymentGateway> {
    repo: R,
    payments: P,
}
```

### ❌ Leaking Driver Errors

```rust
// Bad - Result type exposes sqlx, callers now depend on it
pub fn get_user(id: i32) -> Result<User, sqlx::Error> { todo!() }

// Good - domain error type
pub fn get_user(id: i32) -> Result<User, UserError> { todo!() }
```

### ❌ God Struct

```rust
// Bad - owns everything
pub struct Application {
    db: sqlx::PgPool,
    redis: redis::Client,
    logger: slog::Logger,
    mailer: lettre::SmtpTransport,
    s3: aws_sdk_s3::Client,
    // ... 20 more fields
}

// Good - narrow services composed at the edge (main.rs)
```

---

## Contributing

Add more Rust examples via pull request!
