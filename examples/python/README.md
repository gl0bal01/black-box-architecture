# Python Black Box Examples

Complete before/after examples demonstrating black box architecture transformations in Python.

## Example 1: User Registration Service

### Before (Tightly Coupled)

```python
import bcrypt
import smtplib
from email.mime.text import MIMEText
import psycopg2

class UserManager:
    """God object doing everything - hard to test, hard to change"""

    def __init__(self):
        # Direct dependencies - can't test without real DB and SMTP
        self.db = psycopg2.connect(
            host="localhost",
            database="myapp",
            user="postgres",
            password="secret"
        )
        self.smtp_host = "smtp.gmail.com"
        self.smtp_port = 587

    def register_user(self, email: str, password: str, name: str) -> int:
        """Register a new user - doing too much in one place"""

        # Validation mixed with business logic
        if not email or "@" not in email:
            raise ValueError("Invalid email")

        if len(password) < 8:
            raise ValueError("Password too short")

        # Direct database access
        cursor = self.db.cursor()
        cursor.execute("SELECT id FROM users WHERE email = %s", (email,))
        if cursor.fetchone():
            raise ValueError("Email already exists")

        # Password hashing
        hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt())

        # Insert user
        cursor.execute(
            "INSERT INTO users (email, password, name) VALUES (%s, %s, %s) RETURNING id",
            (email, hashed, name)
        )
        user_id = cursor.fetchone()[0]
        self.db.commit()

        # Send email - hard-coded SMTP logic
        msg = MIMEText("Welcome to our app!")
        msg['Subject'] = "Welcome!"
        msg['From'] = "noreply@example.com"
        msg['To'] = email

        smtp = smtplib.SMTP(self.smtp_host, self.smtp_port)
        smtp.starttls()
        smtp.login("noreply@example.com", "smtp_password")
        smtp.send_message(msg)
        smtp.quit()

        return user_id
```

**Problems:**
- ❌ Can't test without real PostgreSQL database
- ❌ Can't test without real SMTP server
- ❌ Can't swap email provider (locked to SMTP)
- ❌ Can't swap database (locked to PostgreSQL)
- ❌ Validation, storage, and notifications all mixed together
- ❌ One developer can't own this (too complex)
- ❌ Changes cascade (changing email requires touching UserManager)

### After (Black Box Architecture)

```python
from abc import ABC, abstractmethod
from typing import Protocol, Optional
from dataclasses import dataclass

# ===== PRIMITIVES (Core Data Types) =====

@dataclass
class User:
    """Core primitive - represents a user"""
    email: str
    password_hash: str
    name: str
    id: Optional[int] = None


# ===== BLACK BOX INTERFACES =====

class UserRepository(Protocol):
    """Black box interface for user storage - HOW it stores doesn't matter"""

    def find_by_email(self, email: str) -> Optional[User]:
        """Find user by email, return None if not found"""
        ...

    def save(self, user: User) -> int:
        """Save user, return user ID"""
        ...


class EmailService(Protocol):
    """Black box interface for sending emails - implementation hidden"""

    def send_welcome(self, email: str, name: str) -> None:
        """Send welcome email to new user"""
        ...


class PasswordHasher(Protocol):
    """Black box interface for password hashing"""

    def hash(self, password: str) -> str:
        """Hash a password, return hash string"""
        ...

    def verify(self, password: str, hashed: str) -> bool:
        """Verify password against hash"""
        ...


# ===== IMPLEMENTATIONS (Replaceable) =====

class PostgresUserRepository:
    """PostgreSQL implementation - can be replaced with MySQL, MongoDB, etc."""

    def __init__(self, connection):
        self._conn = connection

    def find_by_email(self, email: str) -> Optional[User]:
        cursor = self._conn.cursor()
        cursor.execute("SELECT id, email, password, name FROM users WHERE email = %s", (email,))
        row = cursor.fetchone()
        if not row:
            return None
        return User(id=row[0], email=row[1], password_hash=row[2], name=row[3])

    def save(self, user: User) -> int:
        cursor = self._conn.cursor()
        cursor.execute(
            "INSERT INTO users (email, password, name) VALUES (%s, %s, %s) RETURNING id",
            (user.email, user.password_hash, user.name)
        )
        user_id = cursor.fetchone()[0]
        self._conn.commit()
        return user_id


class BcryptPasswordHasher:
    """Bcrypt implementation - can be replaced with Argon2, etc."""

    def hash(self, password: str) -> str:
        import bcrypt
        return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()

    def verify(self, password: str, hashed: str) -> bool:
        import bcrypt
        return bcrypt.checkpw(password.encode(), hashed.encode())


class SMTPEmailService:
    """SMTP implementation - can be replaced with SendGrid, Mailgun, etc."""

    def __init__(self, host: str, port: int, username: str, password: str):
        self._host = host
        self._port = port
        self._username = username
        self._password = password

    def send_welcome(self, email: str, name: str) -> None:
        import smtplib
        from email.mime.text import MIMEText

        msg = MIMEText(f"Welcome {name}!")
        msg['Subject'] = "Welcome!"
        msg['From'] = self._username
        msg['To'] = email

        with smtplib.SMTP(self._host, self._port) as smtp:
            smtp.starttls()
            smtp.login(self._username, self._password)
            smtp.send_message(msg)


# ===== BUSINESS LOGIC (Single Responsibility) =====

class UserRegistrationService:
    """
    Single responsibility: Register users
    One person can own this
    Easy to understand: just reads the interfaces
    """

    def __init__(
        self,
        user_repo: UserRepository,
        email_service: EmailService,
        password_hasher: PasswordHasher
    ):
        # Dependencies injected, not created
        self._user_repo = user_repo
        self._email_service = email_service
        self._password_hasher = password_hasher

    def register(self, email: str, password: str, name: str) -> int:
        """Register a new user"""

        # Validate
        if not email or "@" not in email:
            raise ValueError("Invalid email")

        if len(password) < 8:
            raise ValueError("Password too short")

        # Check if exists
        if self._user_repo.find_by_email(email):
            raise ValueError("Email already exists")

        # Create user with hashed password
        user = User(
            email=email,
            password_hash=self._password_hasher.hash(password),
            name=name
        )

        # Save
        user_id = self._user_repo.save(user)

        # Send welcome email
        self._email_service.send_welcome(email, name)

        return user_id
```

**Benefits:**
- ✅ **Testable**: Mock interfaces, no real DB/SMTP needed
- ✅ **Replaceable**: Swap PostgreSQL → MySQL, SMTP → SendGrid
- ✅ **Clear**: Each interface does one thing
- ✅ **Maintainable**: One person can own each module
- ✅ **Isolated changes**: Changing email doesn't touch UserRegistrationService

### Testing the Black Box Version

```python
# ===== MOCK IMPLEMENTATIONS FOR TESTING =====

class InMemoryUserRepository:
    """Mock repository for testing - no database needed!"""

    def __init__(self):
        self._users = {}
        self._next_id = 1

    def find_by_email(self, email: str) -> Optional[User]:
        return self._users.get(email)

    def save(self, user: User) -> int:
        user.id = self._next_id
        self._next_id += 1
        self._users[user.email] = user
        return user.id


class MockEmailService:
    """Mock email service for testing - no SMTP needed!"""

    def __init__(self):
        self.sent_emails = []  # Track sent emails

    def send_welcome(self, email: str, name: str) -> None:
        self.sent_emails.append({'email': email, 'name': name})


class FakePasswordHasher:
    """Fake hasher for testing - fast, no bcrypt needed"""

    def hash(self, password: str) -> str:
        return f"hashed_{password}"

    def verify(self, password: str, hashed: str) -> bool:
        return hashed == f"hashed_{password}"


# ===== TESTS =====

import pytest

def test_successful_registration():
    # Arrange - create mocks
    user_repo = InMemoryUserRepository()
    email_service = MockEmailService()
    hasher = FakePasswordHasher()

    service = UserRegistrationService(user_repo, email_service, hasher)

    # Act
    user_id = service.register("test@example.com", "password123", "John")

    # Assert
    assert user_id == 1
    assert len(email_service.sent_emails) == 1
    assert email_service.sent_emails[0]['email'] == "test@example.com"

    # Verify user was saved
    saved_user = user_repo.find_by_email("test@example.com")
    assert saved_user is not None
    assert saved_user.name == "John"
    assert saved_user.password_hash == "hashed_password123"


def test_duplicate_email_rejected():
    user_repo = InMemoryUserRepository()
    email_service = MockEmailService()
    hasher = FakePasswordHasher()

    service = UserRegistrationService(user_repo, email_service, hasher)

    # Register first user
    service.register("test@example.com", "password123", "John")

    # Try to register duplicate
    with pytest.raises(ValueError, match="Email already exists"):
        service.register("test@example.com", "password456", "Jane")


def test_invalid_email_rejected():
    service = UserRegistrationService(
        InMemoryUserRepository(),
        MockEmailService(),
        FakePasswordHasher()
    )

    with pytest.raises(ValueError, match="Invalid email"):
        service.register("not-an-email", "password123", "John")
```

**Testing Benefits:**
- ✅ Tests run in milliseconds (no real DB/SMTP)
- ✅ Tests are reliable (no network dependencies)
- ✅ Easy to test edge cases (duplicate emails, failures, etc.)
- ✅ Can test business logic in isolation

---

## Example 2: Payment Processing

### Before (Tightly Coupled)

```python
import stripe
import requests

class OrderProcessor:
    """Everything mixed together - hard to change payment providers"""

    def __init__(self):
        stripe.api_key = "sk_test_..."  # Hard-coded Stripe

    def process_order(self, amount: float, card_token: str, order_id: int) -> dict:
        # Locked into Stripe - can't easily swap to PayPal or Square
        try:
            charge = stripe.Charge.create(
                amount=int(amount * 100),
                currency="usd",
                source=card_token,
                description=f"Order {order_id}"
            )

            # Update order in database (mixed concerns)
            import psycopg2
            db = psycopg2.connect("...")
            cursor = db.cursor()
            cursor.execute(
                "UPDATE orders SET status = 'paid', transaction_id = %s WHERE id = %s",
                (charge.id, order_id)
            )
            db.commit()

            return {"success": True, "transaction_id": charge.id}
        except stripe.error.CardError as e:
            return {"success": False, "error": str(e)}
```

**Problems:**
- ❌ Locked into Stripe (what if you want PayPal?)
- ❌ Can't test without real Stripe API
- ❌ Payment logic mixed with database logic
- ❌ Hard to add fraud detection or logging

### After (Black Box Architecture)

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Optional

# ===== PRIMITIVES =====

@dataclass
class PaymentResult:
    """Result of a payment attempt"""
    success: bool
    transaction_id: Optional[str] = None
    error: Optional[str] = None


# ===== BLACK BOX INTERFACES =====

class PaymentGateway(Protocol):
    """Black box interface - implementation can be Stripe, PayPal, Square, etc."""

    def charge(self, amount: float, token: str, description: str) -> PaymentResult:
        """Process a payment"""
        ...

    def refund(self, transaction_id: str, amount: float) -> PaymentResult:
        """Refund a payment"""
        ...


class OrderRepository(Protocol):
    """Black box interface for order storage"""

    def update_payment_status(self, order_id: int, transaction_id: str) -> None:
        ...


# ===== IMPLEMENTATIONS (Replaceable) =====

class StripeGateway:
    """Stripe implementation - can be completely replaced"""

    def __init__(self, api_key: str):
        import stripe
        stripe.api_key = api_key
        self._stripe = stripe

    def charge(self, amount: float, token: str, description: str) -> PaymentResult:
        try:
            charge = self._stripe.Charge.create(
                amount=int(amount * 100),
                currency="usd",
                source=token,
                description=description
            )
            return PaymentResult(success=True, transaction_id=charge.id)
        except self._stripe.error.CardError as e:
            return PaymentResult(success=False, error=str(e))

    def refund(self, transaction_id: str, amount: float) -> PaymentResult:
        try:
            refund = self._stripe.Refund.create(
                charge=transaction_id,
                amount=int(amount * 100)
            )
            return PaymentResult(success=True, transaction_id=refund.id)
        except self._stripe.error.StripeError as e:
            return PaymentResult(success=False, error=str(e))


class PayPalGateway:
    """PayPal implementation - completely different internally, same interface"""

    def __init__(self, client_id: str, secret: str):
        self._client_id = client_id
        self._secret = secret

    def charge(self, amount: float, token: str, description: str) -> PaymentResult:
        # PayPal API implementation (completely different from Stripe)
        # But the interface is the same!
        try:
            # Simplified PayPal logic
            response = requests.post(
                "https://api.paypal.com/v1/payments/payment",
                json={
                    "intent": "sale",
                    "payer": {"payment_method": "paypal"},
                    "transactions": [{"amount": {"total": amount, "currency": "USD"}}]
                },
                auth=(self._client_id, self._secret)
            )
            data = response.json()
            return PaymentResult(success=True, transaction_id=data.get('id'))
        except Exception as e:
            return PaymentResult(success=False, error=str(e))

    def refund(self, transaction_id: str, amount: float) -> PaymentResult:
        # PayPal refund implementation
        return PaymentResult(success=True, transaction_id=f"refund_{transaction_id}")


# ===== BUSINESS LOGIC =====

class OrderProcessor:
    """
    Single responsibility: Process orders
    Works with ANY payment gateway!
    """

    def __init__(self, payment_gateway: PaymentGateway, order_repo: OrderRepository):
        self._gateway = payment_gateway
        self._order_repo = order_repo

    def process_order(self, amount: float, token: str, order_id: int) -> PaymentResult:
        """Process an order payment"""

        # Process payment (doesn't care if it's Stripe or PayPal!)
        result = self._gateway.charge(amount, token, f"Order {order_id}")

        # Update order if successful
        if result.success:
            self._order_repo.update_payment_status(order_id, result.transaction_id)

        return result
```

**Benefits:**
- ✅ **Swap payment providers**: Change one line, use PayPal instead of Stripe
- ✅ **A/B test**: Try Stripe for some customers, PayPal for others
- ✅ **Easy testing**: Mock payment gateway, no real charges
- ✅ **Gradual migration**: Switch providers one route at a time

### Usage Example

```python
# Use Stripe
processor = OrderProcessor(
    payment_gateway=StripeGateway("sk_test_..."),
    order_repo=PostgresOrderRepository(conn)
)

# Or use PayPal (just change one line!)
processor = OrderProcessor(
    payment_gateway=PayPalGateway("client_id", "secret"),
    order_repo=PostgresOrderRepository(conn)
)

# Business logic unchanged
result = processor.process_order(99.99, "tok_visa", order_id=123)
```

---

## Key Patterns in Python

### 1. Protocol (Python 3.8+) - Duck Typing with Type Safety

```python
from typing import Protocol

class Storage(Protocol):
    """Define interface without inheritance"""

    def save(self, key: str, data: bytes) -> None: ...
    def load(self, key: str) -> bytes: ...

# Any class with these methods satisfies the protocol
class FileStorage:
    def save(self, key: str, data: bytes) -> None:
        with open(f"/data/{key}", "wb") as f:
            f.write(data)

    def load(self, key: str) -> bytes:
        with open(f"/data/{key}", "rb") as f:
            return f.read()

# Works! FileStorage satisfies Storage protocol
def use_storage(storage: Storage):
    storage.save("test", b"data")
```

### 2. Abstract Base Classes - Explicit Interfaces

```python
from abc import ABC, abstractmethod

class Cache(ABC):
    """Explicit interface with enforcement"""

    @abstractmethod
    def get(self, key: str) -> Optional[str]:
        pass

    @abstractmethod
    def set(self, key: str, value: str, ttl: int) -> None:
        pass

class RedisCache(Cache):
    def get(self, key: str) -> Optional[str]:
        # Implementation
        pass

    def set(self, key: str, value: str, ttl: int) -> None:
        # Implementation
        pass
```

### 3. Dependency Injection

```python
# Constructor injection (preferred)
class UserService:
    def __init__(self, repo: UserRepository, email: EmailService):
        self._repo = repo
        self._email = email

# Setter injection (when needed)
class ConfigManager:
    def set_storage(self, storage: Storage) -> None:
        self._storage = storage
```

---

## Replaceability Test

Can you rewrite an implementation using only the interface docs?

```python
# ===== INTERFACE DOCUMENTATION =====

class EmailService(Protocol):
    """
    Send emails to users.

    Methods:
        send_welcome(email, name): Send welcome email to new user
    """
    def send_welcome(self, email: str, name: str) -> None: ...


# ===== IMPLEMENTATION 1: SMTP =====

class SMTPEmailService:
    """Send emails via SMTP server"""
    # Implementation using smtplib
    pass


# ===== IMPLEMENTATION 2: SendGrid =====

class SendGridEmailService:
    """Send emails via SendGrid API"""
    # Completely different implementation
    # But satisfies the same interface!
    pass


# ===== IMPLEMENTATION 3: Mock (Testing) =====

class MockEmailService:
    """Mock for testing - no real emails sent"""
    def __init__(self):
        self.sent = []

    def send_welcome(self, email: str, name: str) -> None:
        self.sent.append({'email': email, 'name': name})
```

All three implementations satisfy `EmailService` interface. Business logic works with any of them!

---

## Common Anti-Patterns to Avoid

### ❌ Leaky Abstraction

```python
class UserRepository(Protocol):
    def save(self, user: User) -> int: ...
    def get_database_connection(self) -> Connection: ...  # ❌ Leaks implementation!
```

### ❌ God Interface

```python
class UserService(Protocol):
    # ❌ Too many responsibilities
    def register(self): ...
    def login(self): ...
    def update_profile(self): ...
    def send_notification(self): ...
    def process_payment(self): ...
```

### ❌ Tight Coupling

```python
class OrderService:
    def __init__(self):
        self.payment = StripeGateway()  # ❌ Hard-coded dependency
```

---

## Contributing

Add more Python examples via pull request!
