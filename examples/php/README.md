# PHP Black Box Examples

Black box architecture patterns in PHP using interfaces, dependency injection, and design patterns.

## Key Patterns

### 1. Interface-Based Design

**Before (Tightly Coupled)**:
```php
<?php
class UserService {
    public function getUser($id) {
        // Direct database access
        $pdo = new PDO('mysql:host=localhost;dbname=mydb', 'user', 'pass');
        $stmt = $pdo->prepare('SELECT * FROM users WHERE id = ?');
        $stmt->execute([$id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }

    public function saveUser($user) {
        $pdo = new PDO('mysql:host=localhost;dbname=mydb', 'user', 'pass');
        $stmt = $pdo->prepare('INSERT INTO users (name, email) VALUES (?, ?)');
        $stmt->execute([$user['name'], $user['email']]);
    }
}
```

**After (Black Box with Interface)**:
```php
<?php
// Black box interface
interface UserRepositoryInterface {
    public function find(int $id): ?array;
    public function save(array $user): void;
    public function delete(int $id): void;
}

// Implementation hidden
class MySQLUserRepository implements UserRepositoryInterface {
    private PDO $pdo;

    public function __construct(PDO $pdo) {
        $this->pdo = $pdo;
    }

    public function find(int $id): ?array {
        $stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = ?');
        $stmt->execute([$id]);
        $result = $stmt->fetch(PDO::FETCH_ASSOC);
        return $result ?: null;
    }

    public function save(array $user): void {
        $stmt = $this->pdo->prepare(
            'INSERT INTO users (name, email) VALUES (?, ?)
             ON DUPLICATE KEY UPDATE name = ?, email = ?'
        );
        $stmt->execute([
            $user['name'],
            $user['email'],
            $user['name'],
            $user['email']
        ]);
    }

    public function delete(int $id): void {
        $stmt = $this->pdo->prepare('DELETE FROM users WHERE id = ?');
        $stmt->execute([$id]);
    }
}

// Service uses interface only
class UserService {
    private UserRepositoryInterface $repository;

    public function __construct(UserRepositoryInterface $repository) {
        $this->repository = $repository;
    }

    public function getUser(int $id): ?array {
        return $this->repository->find($id);
    }

    public function createUser(string $name, string $email): void {
        $this->repository->save(['name' => $name, 'email' => $email]);
    }
}
```

**Benefits**:
- ✅ Can swap database (MySQL → PostgreSQL → MongoDB)
- ✅ Easy to test with mock repository
- ✅ No SQL in business logic
- ✅ Database changes don't affect UserService

---

### 2. Dependency Injection Container

**Before (Hard-Coded Dependencies)**:
```php
<?php
class OrderController {
    public function createOrder() {
        // Hard-coded dependencies
        $pdo = new PDO('mysql:host=localhost;dbname=mydb', 'user', 'pass');
        $orderRepo = new MySQLOrderRepository($pdo);
        $emailService = new SMTPEmailService('smtp.example.com');

        $order = $orderRepo->create($_POST);
        $emailService->send($_POST['email'], 'Order created', 'Thank you!');
    }
}
```

**After (Black Box with DI)**:
```php
<?php
// Email interface
interface EmailServiceInterface {
    public function send(string $to, string $subject, string $body): void;
}

// Order repository interface
interface OrderRepositoryInterface {
    public function create(array $data): int;
    public function find(int $id): ?array;
}

// SMTP implementation
class SMTPEmailService implements EmailServiceInterface {
    private string $host;

    public function __construct(string $host) {
        $this->host = $host;
    }

    public function send(string $to, string $subject, string $body): void {
        // SMTP implementation
        mail($to, $subject, $body);
    }
}

// Alternative: SendGrid implementation
class SendGridEmailService implements EmailServiceInterface {
    private string $apiKey;

    public function __construct(string $apiKey) {
        $this->apiKey = $apiKey;
    }

    public function send(string $to, string $subject, string $body): void {
        // SendGrid API implementation
    }
}

// Controller with injected dependencies
class OrderController {
    private OrderRepositoryInterface $orderRepo;
    private EmailServiceInterface $emailService;

    public function __construct(
        OrderRepositoryInterface $orderRepo,
        EmailServiceInterface $emailService
    ) {
        $this->orderRepo = $orderRepo;
        $this->emailService = $emailService;
    }

    public function createOrder(array $data): void {
        $orderId = $this->orderRepo->create($data);
        $this->emailService->send(
            $data['email'],
            'Order Created',
            "Your order #$orderId has been created"
        );
    }
}
```

**Usage with DI Container**:
```php
<?php
// Simple DI container
class Container {
    private array $bindings = [];

    public function bind(string $interface, callable $factory): void {
        $this->bindings[$interface] = $factory;
    }

    public function make(string $interface) {
        if (!isset($this->bindings[$interface])) {
            throw new Exception("No binding for $interface");
        }
        return $this->bindings[$interface]($this);
    }
}

// Bootstrap
$container = new Container();

// Bind interfaces to implementations
$container->bind(OrderRepositoryInterface::class, function($c) {
    $pdo = new PDO('mysql:host=localhost;dbname=mydb', 'user', 'pass');
    return new MySQLOrderRepository($pdo);
});

$container->bind(EmailServiceInterface::class, function($c) {
    // Easy to swap: return new SendGridEmailService($_ENV['SENDGRID_KEY']);
    return new SMTPEmailService('smtp.example.com');
});

// Create controller with dependencies
$controller = new OrderController(
    $container->make(OrderRepositoryInterface::class),
    $container->make(EmailServiceInterface::class)
);

$controller->createOrder($_POST);
```

---

### 3. Strategy Pattern (Pluggable Algorithms)

**Before (Hard-Coded Logic)**:
```php
<?php
class PaymentProcessor {
    public function process($amount, $method) {
        if ($method === 'stripe') {
            // Stripe processing
            $stripe = new \Stripe\StripeClient($_ENV['STRIPE_KEY']);
            return $stripe->charges->create(['amount' => $amount]);
        } elseif ($method === 'paypal') {
            // PayPal processing
            $paypal = new \PayPal\Rest\ApiContext();
            // ...
        } elseif ($method === 'bank') {
            // Bank transfer logic
        }
    }
}
```

**After (Black Box Strategy)**:
```php
<?php
// Payment strategy interface
interface PaymentStrategyInterface {
    public function charge(float $amount): array;
    public function refund(string $transactionId): bool;
}

// Stripe strategy
class StripePaymentStrategy implements PaymentStrategyInterface {
    private \Stripe\StripeClient $client;

    public function __construct(string $apiKey) {
        $this->client = new \Stripe\StripeClient($apiKey);
    }

    public function charge(float $amount): array {
        $charge = $this->client->charges->create([
            'amount' => $amount * 100, // Cents
            'currency' => 'usd'
        ]);

        return [
            'transaction_id' => $charge->id,
            'status' => $charge->status
        ];
    }

    public function refund(string $transactionId): bool {
        $refund = $this->client->refunds->create([
            'charge' => $transactionId
        ]);
        return $refund->status === 'succeeded';
    }
}

// PayPal strategy
class PayPalPaymentStrategy implements PaymentStrategyInterface {
    private \PayPal\Rest\ApiContext $context;

    public function __construct(\PayPal\Rest\ApiContext $context) {
        $this->context = $context;
    }

    public function charge(float $amount): array {
        // PayPal implementation
        return ['transaction_id' => 'pp_xxx', 'status' => 'completed'];
    }

    public function refund(string $transactionId): bool {
        // PayPal refund implementation
        return true;
    }
}

// Payment processor uses strategy
class PaymentProcessor {
    private PaymentStrategyInterface $strategy;

    public function __construct(PaymentStrategyInterface $strategy) {
        $this->strategy = $strategy;
    }

    public function setStrategy(PaymentStrategyInterface $strategy): void {
        $this->strategy = $strategy;
    }

    public function processPayment(float $amount): array {
        return $this->strategy->charge($amount);
    }

    public function refundPayment(string $transactionId): bool {
        return $this->strategy->refund($transactionId);
    }
}
```

**Usage**:
```php
<?php
// Use Stripe
$processor = new PaymentProcessor(
    new StripePaymentStrategy($_ENV['STRIPE_KEY'])
);
$result = $processor->processPayment(99.99);

// Easily switch to PayPal
$processor->setStrategy(
    new PayPalPaymentStrategy($paypalContext)
);
$result = $processor->processPayment(99.99);
```

---

### 4. Service Layer Pattern

**Before (Fat Controllers)**:
```php
<?php
class UserController {
    public function register() {
        // Validation
        if (!filter_var($_POST['email'], FILTER_VALIDATE_EMAIL)) {
            throw new Exception('Invalid email');
        }

        // Business logic
        $pdo = new PDO('...');
        $stmt = $pdo->prepare('SELECT * FROM users WHERE email = ?');
        $stmt->execute([$_POST['email']]);
        if ($stmt->fetch()) {
            throw new Exception('Email exists');
        }

        // Password hashing
        $hash = password_hash($_POST['password'], PASSWORD_BCRYPT);

        // Database insert
        $stmt = $pdo->prepare('INSERT INTO users (email, password) VALUES (?, ?)');
        $stmt->execute([$_POST['email'], $hash]);

        // Send email
        mail($_POST['email'], 'Welcome', 'Thanks for registering!');

        // Log
        error_log("User registered: {$_POST['email']}");
    }
}
```

**After (Black Box Service Layer)**:
```php
<?php
// Validator interface
interface ValidatorInterface {
    public function validate(array $data): array;
}

// Email validator
class EmailValidator implements ValidatorInterface {
    public function validate(array $data): array {
        $errors = [];
        if (!filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
            $errors['email'] = 'Invalid email address';
        }
        return $errors;
    }
}

// User service encapsulates business logic
class UserRegistrationService {
    private UserRepositoryInterface $userRepo;
    private EmailServiceInterface $emailService;
    private ValidatorInterface $validator;
    private LoggerInterface $logger;

    public function __construct(
        UserRepositoryInterface $userRepo,
        EmailServiceInterface $emailService,
        ValidatorInterface $validator,
        LoggerInterface $logger
    ) {
        $this->userRepo = $userRepo;
        $this->emailService = $emailService;
        $this->validator = $validator;
        $this->logger = $logger;
    }

    public function register(array $data): array {
        // Validate
        $errors = $this->validator->validate($data);
        if (!empty($errors)) {
            return ['success' => false, 'errors' => $errors];
        }

        // Check if exists
        if ($this->userRepo->findByEmail($data['email'])) {
            return ['success' => false, 'errors' => ['email' => 'Email already exists']];
        }

        // Hash password
        $data['password'] = password_hash($data['password'], PASSWORD_BCRYPT);

        // Save user
        $userId = $this->userRepo->save($data);

        // Send welcome email
        $this->emailService->send(
            $data['email'],
            'Welcome!',
            'Thanks for registering!'
        );

        // Log
        $this->logger->info("User registered", ['email' => $data['email']]);

        return ['success' => true, 'user_id' => $userId];
    }
}

// Thin controller
class UserController {
    private UserRegistrationService $registrationService;

    public function __construct(UserRegistrationService $registrationService) {
        $this->registrationService = $registrationService;
    }

    public function register() {
        $result = $this->registrationService->register($_POST);

        if ($result['success']) {
            http_response_code(201);
            echo json_encode(['message' => 'Registration successful']);
        } else {
            http_response_code(400);
            echo json_encode(['errors' => $result['errors']]);
        }
    }
}
```

---

## Testing with Black Boxes

```php
<?php
use PHPUnit\Framework\TestCase;

class UserRegistrationServiceTest extends TestCase {
    public function testSuccessfulRegistration() {
        // Mock dependencies
        $userRepo = $this->createMock(UserRepositoryInterface::class);
        $userRepo->method('findByEmail')->willReturn(null);
        $userRepo->method('save')->willReturn(123);

        $emailService = $this->createMock(EmailServiceInterface::class);
        $validator = $this->createMock(ValidatorInterface::class);
        $validator->method('validate')->willReturn([]);
        $logger = $this->createMock(LoggerInterface::class);

        // Create service with mocks
        $service = new UserRegistrationService(
            $userRepo,
            $emailService,
            $validator,
            $logger
        );

        // Test
        $result = $service->register([
            'email' => 'test@example.com',
            'password' => 'password123'
        ]);

        $this->assertTrue($result['success']);
        $this->assertEquals(123, $result['user_id']);
    }

    public function testDuplicateEmail() {
        $userRepo = $this->createMock(UserRepositoryInterface::class);
        $userRepo->method('findByEmail')->willReturn(['id' => 1]);

        $service = new UserRegistrationService(
            $userRepo,
            $this->createMock(EmailServiceInterface::class),
            new EmailValidator(),
            $this->createMock(LoggerInterface::class)
        );

        $result = $service->register([
            'email' => 'existing@example.com',
            'password' => 'password123'
        ]);

        $this->assertFalse($result['success']);
        $this->assertArrayHasKey('errors', $result);
    }
}
```

---

## Laravel Example (Modern PHP Framework)

```php
<?php
namespace App\Services;

// Interface
interface PaymentGatewayInterface {
    public function charge(float $amount, string $token): array;
}

// Stripe implementation
class StripeGateway implements PaymentGatewayInterface {
    public function charge(float $amount, string $token): array {
        // Stripe logic
        return ['transaction_id' => 'stripe_xxx', 'status' => 'success'];
    }
}

// PayPal implementation
class PayPalGateway implements PaymentGatewayInterface {
    public function charge(float $amount, string $token): array {
        // PayPal logic
        return ['transaction_id' => 'pp_xxx', 'status' => 'success'];
    }
}

// Service provider (Laravel DI)
class AppServiceProvider extends ServiceProvider {
    public function register() {
        // Bind interface to implementation
        $this->app->bind(PaymentGatewayInterface::class, function ($app) {
            // Can easily switch based on config
            return config('payment.gateway') === 'stripe'
                ? new StripeGateway()
                : new PayPalGateway();
        });
    }
}

// Controller
class PaymentController extends Controller {
    private PaymentGatewayInterface $gateway;

    public function __construct(PaymentGatewayInterface $gateway) {
        $this->gateway = $gateway;
    }

    public function charge(Request $request) {
        $result = $this->gateway->charge(
            $request->input('amount'),
            $request->input('token')
        );

        return response()->json($result);
    }
}
```

---

## Common Patterns

### Pattern 1: Repository + Service
- **Repository**: Data access
- **Service**: Business logic
- **Controller**: HTTP handling

### Pattern 2: Factory Pattern
```php
interface LoggerInterface {
    public function log(string $message): void;
}

class LoggerFactory {
    public static function create(string $type): LoggerInterface {
        return match($type) {
            'file' => new FileLogger(),
            'database' => new DatabaseLogger(),
            'syslog' => new SyslogLogger(),
            default => throw new InvalidArgumentException()
        };
    }
}
```

### Pattern 3: Adapter Pattern
```php
// Wrap third-party library
interface CacheInterface {
    public function get(string $key): mixed;
    public function set(string $key, mixed $value, int $ttl): void;
}

class RedisAdapter implements CacheInterface {
    private Redis $redis;

    public function get(string $key): mixed {
        return $this->redis->get($key);
    }

    public function set(string $key, mixed $value, int $ttl): void {
        $this->redis->setex($key, $ttl, serialize($value));
    }
}
```

---

## PSR Standards

PHP has PSR (PHP Standards Recommendations) for black box interfaces:

- **PSR-3**: Logger Interface
- **PSR-6**: Caching Interface
- **PSR-7**: HTTP Message Interface
- **PSR-11**: Container Interface
- **PSR-15**: HTTP Server Request Handlers
- **PSR-18**: HTTP Client Interface

These are industry-standard black box interfaces!

---

## Contributing

Add more PHP examples via pull request!
