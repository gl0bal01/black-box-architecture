# TypeScript Black Box Examples

Complete before/after examples demonstrating black box architecture transformations in TypeScript.

## Example 1: User Authentication System

### Before (Tightly Coupled)

```typescript
import bcrypt from 'bcrypt';
import nodemailer from 'nodemailer';
import { Pool } from 'pg';

class AuthManager {
  // Hard-coded dependencies - can't test without real DB and SMTP
  private db = new Pool({
    host: 'localhost',
    database: 'myapp',
    user: 'postgres',
    password: 'secret'
  });

  private mailer = nodemailer.createTransport({
    host: 'smtp.gmail.com',
    port: 587,
    auth: { user: 'noreply@example.com', pass: 'password' }
  });

  async registerUser(email: string, password: string): Promise<number> {
    // Validation mixed with everything else
    if (!email.includes('@')) {
      throw new Error('Invalid email');
    }

    // Direct database access
    const existing = await this.db.query(
      'SELECT id FROM users WHERE email = $1',
      [email]
    );

    if (existing.rows.length > 0) {
      throw new Error('Email exists');
    }

    // Password hashing
    const hash = await bcrypt.hash(password, 10);

    // Insert user
    const result = await this.db.query(
      'INSERT INTO users (email, password) VALUES ($1, $2) RETURNING id',
      [email, hash]
    );

    const userId = result.rows[0].id;

    // Send email - hard-coded mailer
    await this.mailer.sendMail({
      from: 'noreply@example.com',
      to: email,
      subject: 'Welcome!',
      text: 'Thanks for signing up'
    });

    return userId;
  }
}
```

**Problems:**
- ❌ Can't test without PostgreSQL
- ❌ Can't test without SMTP server
- ❌ Can't swap email provider
- ❌ Can't swap database
- ❌ Everything mixed together
- ❌ Changes cascade

### After (Black Box Architecture)

```typescript
// ===== PRIMITIVES (Core Data Types) =====

interface User {
  id?: number;
  email: string;
  passwordHash: string;
}

// ===== BLACK BOX INTERFACES =====

interface UserRepository {
  /**
   * Find user by email
   * HOW it finds doesn't matter - could be PostgreSQL, MySQL, MongoDB
   */
  findByEmail(email: string): Promise<User | null>;

  /**
   * Save a new user
   * Returns the user ID
   */
  save(user: User): Promise<number>;
}

interface EmailService {
  /**
   * Send welcome email
   * Implementation hidden - could be SMTP, SendGrid, Mailgun
   */
  sendWelcome(email: string): Promise<void>;
}

interface PasswordHasher {
  /**
   * Hash a password
   */
  hash(password: string): Promise<string>;

  /**
   * Verify password against hash
   */
  verify(password: string, hash: string): Promise<boolean>;
}

// ===== IMPLEMENTATIONS (Replaceable) =====

class PostgresUserRepository implements UserRepository {
  constructor(private pool: Pool) {}

  async findByEmail(email: string): Promise<User | null> {
    const result = await this.pool.query(
      'SELECT id, email, password FROM users WHERE email = $1',
      [email]
    );

    if (result.rows.length === 0) return null;

    return {
      id: result.rows[0].id,
      email: result.rows[0].email,
      passwordHash: result.rows[0].password
    };
  }

  async save(user: User): Promise<number> {
    const result = await this.pool.query(
      'INSERT INTO users (email, password) VALUES ($1, $2) RETURNING id',
      [user.email, user.passwordHash]
    );
    return result.rows[0].id;
  }
}

class BcryptHasher implements PasswordHasher {
  async hash(password: string): Promise<string> {
    return bcrypt.hash(password, 10);
  }

  async verify(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
  }
}

class SMTPEmailService implements EmailService {
  constructor(private transporter: nodemailer.Transporter) {}

  async sendWelcome(email: string): Promise<void> {
    await this.transporter.sendMail({
      from: 'noreply@example.com',
      to: email,
      subject: 'Welcome!',
      text: 'Thanks for signing up'
    });
  }
}

// ===== BUSINESS LOGIC (Single Responsibility) =====

class UserRegistrationService {
  constructor(
    private userRepo: UserRepository,
    private emailService: EmailService,
    private hasher: PasswordHasher
  ) {}

  async register(email: string, password: string): Promise<number> {
    // Validate
    if (!email.includes('@')) {
      throw new Error('Invalid email');
    }

    // Check if exists
    const existing = await this.userRepo.findByEmail(email);
    if (existing) {
      throw new Error('Email exists');
    }

    // Create user
    const user: User = {
      email,
      passwordHash: await this.hasher.hash(password)
    };

    // Save
    const userId = await this.userRepo.save(user);

    // Send welcome email
    await this.emailService.sendWelcome(email);

    return userId;
  }
}
```

**Benefits:**
- ✅ **Testable**: Mock all dependencies
- ✅ **Replaceable**: Swap implementations easily
- ✅ **Clear**: Each interface has one job
- ✅ **Maintainable**: Simple, focused modules
- ✅ **Type-safe**: Full TypeScript inference

### Testing the Black Box Version

```typescript
// ===== MOCK IMPLEMENTATIONS FOR TESTING =====

class InMemoryUserRepository implements UserRepository {
  private users = new Map<string, User>();
  private nextId = 1;

  async findByEmail(email: string): Promise<User | null> {
    return this.users.get(email) ?? null;
  }

  async save(user: User): Promise<number> {
    const id = this.nextId++;
    this.users.set(user.email, { ...user, id });
    return id;
  }
}

class MockEmailService implements EmailService {
  sentEmails: string[] = [];

  async sendWelcome(email: string): Promise<void> {
    this.sentEmails.push(email);
  }
}

class FakeHasher implements PasswordHasher {
  async hash(password: string): Promise<string> {
    return `hashed_${password}`;
  }

  async verify(password: string, hash: string): Promise<boolean> {
    return hash === `hashed_${password}`;
  }
}

// ===== TESTS =====

describe('UserRegistrationService', () => {
  it('should register a new user successfully', async () => {
    // Arrange
    const userRepo = new InMemoryUserRepository();
    const emailService = new MockEmailService();
    const hasher = new FakeHasher();

    const service = new UserRegistrationService(userRepo, emailService, hasher);

    // Act
    const userId = await service.register('test@example.com', 'password123');

    // Assert
    expect(userId).toBe(1);
    expect(emailService.sentEmails).toContain('test@example.com');

    const savedUser = await userRepo.findByEmail('test@example.com');
    expect(savedUser).toBeDefined();
    expect(savedUser?.passwordHash).toBe('hashed_password123');
  });

  it('should reject duplicate email', async () => {
    const service = new UserRegistrationService(
      new InMemoryUserRepository(),
      new MockEmailService(),
      new FakeHasher()
    );

    await service.register('test@example.com', 'password123');

    await expect(
      service.register('test@example.com', 'password456')
    ).rejects.toThrow('Email exists');
  });
});
```

---

## Example 2: API Client with Retry Logic

### Before (Tightly Coupled)

```typescript
import axios from 'axios';

class ProductService {
  async getProduct(id: string): Promise<Product> {
    // Locked into axios - can't easily swap HTTP library
    let retries = 3;
    let lastError: Error | null = null;

    while (retries > 0) {
      try {
        const response = await axios.get(`https://api.example.com/products/${id}`, {
          headers: { 'Authorization': `Bearer ${process.env.API_KEY}` }
        });

        // Caching mixed with API logic
        const redis = new Redis();
        await redis.set(`product:${id}`, JSON.stringify(response.data), 'EX', 3600);

        return response.data;
      } catch (error) {
        lastError = error as Error;
        retries--;
        await new Promise(r => setTimeout(r, 1000));
      }
    }

    throw lastError!;
  }
}
```

**Problems:**
- ❌ Can't test without real API
- ❌ Can't swap HTTP client
- ❌ Retry logic mixed with business logic
- ❌ Caching mixed with API calls
- ❌ Hard-coded delays

### After (Black Box Architecture)

```typescript
// ===== PRIMITIVES =====

interface Product {
  id: string;
  name: string;
  price: number;
}

interface FetchResult<T> {
  success: boolean;
  data?: T;
  error?: string;
}

// ===== BLACK BOX INTERFACES =====

interface HttpClient {
  /**
   * Make GET request
   * Implementation hidden - could use axios, fetch, or custom client
   */
  get<T>(url: string, headers?: Record<string, string>): Promise<FetchResult<T>>;
}

interface Cache {
  /**
   * Get cached value
   */
  get<T>(key: string): Promise<T | null>;

  /**
   * Set cached value with TTL
   */
  set<T>(key: string, value: T, ttlSeconds: number): Promise<void>;
}

interface RetryStrategy {
  /**
   * Execute function with retry logic
   */
  execute<T>(fn: () => Promise<T>): Promise<T>;
}

// ===== IMPLEMENTATIONS (Replaceable) =====

class AxiosHttpClient implements HttpClient {
  async get<T>(url: string, headers?: Record<string, string>): Promise<FetchResult<T>> {
    try {
      const response = await axios.get(url, { headers });
      return { success: true, data: response.data };
    } catch (error) {
      return { success: false, error: (error as Error).message };
    }
  }
}

class RedisCache implements Cache {
  constructor(private redis: Redis) {}

  async get<T>(key: string): Promise<T | null> {
    const data = await this.redis.get(key);
    return data ? JSON.parse(data) : null;
  }

  async set<T>(key: string, value: T, ttlSeconds: number): Promise<void> {
    await this.redis.set(key, JSON.stringify(value), 'EX', ttlSeconds);
  }
}

class ExponentialBackoffRetry implements RetryStrategy {
  constructor(
    private maxRetries: number = 3,
    private baseDelay: number = 1000
  ) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    let lastError: Error | null = null;

    for (let attempt = 0; attempt < this.maxRetries; attempt++) {
      try {
        return await fn();
      } catch (error) {
        lastError = error as Error;
        if (attempt < this.maxRetries - 1) {
          const delay = this.baseDelay * Math.pow(2, attempt);
          await new Promise(r => setTimeout(r, delay));
        }
      }
    }

    throw lastError!;
  }
}

// ===== BUSINESS LOGIC =====

class ProductService {
  constructor(
    private httpClient: HttpClient,
    private cache: Cache,
    private retry: RetryStrategy,
    private apiKey: string
  ) {}

  async getProduct(id: string): Promise<Product> {
    // Check cache first
    const cached = await this.cache.get<Product>(`product:${id}`);
    if (cached) return cached;

    // Fetch with retry
    const product = await this.retry.execute(async () => {
      const result = await this.httpClient.get<Product>(
        `https://api.example.com/products/${id}`,
        { Authorization: `Bearer ${this.apiKey}` }
      );

      if (!result.success) {
        throw new Error(result.error);
      }

      return result.data!;
    });

    // Cache result
    await this.cache.set(`product:${id}`, product, 3600);

    return product;
  }
}
```

**Benefits:**
- ✅ **Testable**: Mock HTTP, cache, retry independently
- ✅ **Replaceable**: Swap axios → fetch, Redis → Memcached
- ✅ **Flexible**: Change retry strategy (linear, exponential, circuit breaker)
- ✅ **Clear separation**: HTTP / Cache / Retry all isolated

### Testing Example

```typescript
class MockHttpClient implements HttpClient {
  constructor(private mockData: any) {}

  async get<T>(): Promise<FetchResult<T>> {
    return { success: true, data: this.mockData };
  }
}

class InMemoryCache implements Cache {
  private store = new Map<string, any>();

  async get<T>(key: string): Promise<T | null> {
    return this.store.get(key) ?? null;
  }

  async set<T>(key: string, value: T): Promise<void> {
    this.store.set(key, value);
  }
}

class NoRetry implements RetryStrategy {
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    return fn();
  }
}

describe('ProductService', () => {
  it('should fetch and cache product', async () => {
    const mockProduct = { id: '1', name: 'Widget', price: 9.99 };

    const service = new ProductService(
      new MockHttpClient(mockProduct),
      new InMemoryCache(),
      new NoRetry(),
      'test-key'
    );

    const product = await service.getProduct('1');

    expect(product).toEqual(mockProduct);
  });
});
```

---

## Key Patterns in TypeScript

### 1. Interface-First Design

```typescript
// Define WHAT before HOW
interface Logger {
  info(message: string): void;
  error(message: string, error?: Error): void;
}

// Multiple implementations
class ConsoleLogger implements Logger {
  info(message: string): void {
    console.log(message);
  }

  error(message: string, error?: Error): void {
    console.error(message, error);
  }
}

class FileLogger implements Logger {
  constructor(private filepath: string) {}

  info(message: string): void {
    // Write to file
  }

  error(message: string, error?: Error): void {
    // Write to file
  }
}
```

### 2. Dependency Injection with Constructors

```typescript
class OrderService {
  constructor(
    private orderRepo: OrderRepository,
    private payment: PaymentGateway,
    private notifications: NotificationService,
    private logger: Logger
  ) {}

  async createOrder(data: OrderData): Promise<Order> {
    this.logger.info('Creating order');

    const order = await this.orderRepo.create(data);
    await this.payment.charge(order.total);
    await this.notifications.sendConfirmation(order.customerEmail);

    return order;
  }
}
```

### 3. Generic Interfaces

```typescript
interface Repository<T> {
  findById(id: string): Promise<T | null>;
  save(entity: T): Promise<T>;
  delete(id: string): Promise<void>;
}

class UserRepository implements Repository<User> {
  async findById(id: string): Promise<User | null> {
    // Implementation
    return null;
  }

  async save(user: User): Promise<User> {
    // Implementation
    return user;
  }

  async delete(id: string): Promise<void> {
    // Implementation
  }
}
```

---

## Factory Pattern for Swapping Implementations

```typescript
interface StorageFactory {
  createStorage(): Storage;
}

class LocalStorageFactory implements StorageFactory {
  createStorage(): Storage {
    return new FileStorage('./data');
  }
}

class CloudStorageFactory implements StorageFactory {
  createStorage(): Storage {
    return new S3Storage(process.env.AWS_BUCKET!);
  }
}

// Easy to swap
const factory = process.env.NODE_ENV === 'production'
  ? new CloudStorageFactory()
  : new LocalStorageFactory();

const storage = factory.createStorage();
```

---

## Common Anti-Patterns to Avoid

### ❌ Leaky Abstraction

```typescript
interface UserRepository {
  findById(id: string): Promise<User>;
  getMongoClient(): MongoClient; // ❌ Exposes implementation!
}
```

### ❌ God Class

```typescript
class AppService {
  // ❌ Too many responsibilities
  async registerUser() {}
  async processPayment() {}
  async sendEmail() {}
  async generateReport() {}
  async uploadFile() {}
}
```

### ❌ Concrete Dependencies

```typescript
class OrderService {
  private stripe = new Stripe(process.env.STRIPE_KEY); // ❌ Hard-coded

  constructor() {} // ❌ No injection
}
```

---

## Contributing

Add more TypeScript examples via pull request!
