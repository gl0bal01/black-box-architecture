# Black Box Architecture Principles

This document explains the core principles from Eskil Steenberg's [lecture on architecting large software projects](https://www.youtube.com/watch?v=sSpULGNHyoI).

## The Core Philosophy

**"It's faster to write 5 lines of code today than to write 1 line today and then have to edit it in the future."**

Traditional software development slows down as codebases grow. Black box architecture maintains **constant developer velocity** regardless of project size.

---

## The Five Pillars

### 1. Primitive-First Design

**Concept**: Identify the fundamental "atoms" of your system

**Why It Matters**: Everything is built from these core types. Get them right, and the rest follows naturally.

**Examples**:
- **Unix**: Everything is a file
- **Graphics Engine**: Everything is a polygon
- **Chat App**: Messages and Users
- **E-commerce**: Products, Orders, Payments

**How to Identify Primitives**:
- What data flows through your system?
- What operations are performed on this data?
- Can you build complex features by composing these simple types?

**Good Primitive Checklist**:
- ✅ Simple (< 5 min to explain)
- ✅ Consistent across the system
- ✅ Composable (complex from simple)
- ✅ Stable (unlikely to change fundamentally)
- ✅ General (handles current + future needs)

---

### 2. Black Box Interfaces

**Concept**: Modules communicate only through documented interfaces. Implementation details are completely hidden.

**Why It Matters**: You can understand and use a module without knowing how it works internally.

**The Test**: Can you describe what it does without saying how?

**Example**:

**Black Box (Good)**:
```typescript
interface UserAuthenticator {
  // What it does (not how)
  authenticate(email: string, password: string): AuthResult;
}
```

**Leaky Abstraction (Bad)**:
```typescript
interface UserAuthenticator {
  authenticate(email: string, password: string): AuthResult;
  getPasswordHashingAlgorithm(): string;  // ❌ Exposes implementation
  getDatabaseConnection(): Connection;     // ❌ Exposes implementation
}
```

**Benefits**:
- Easy to test (mock the interface)
- Easy to replace (rewrite implementation)
- Easy to understand (just read the interface)
- Changes don't cascade (internal changes invisible)

---

### 3. Replaceable Components

**Concept**: Any module can be completely rewritten from scratch using only its interface documentation

**Why It Matters**: If you can't understand it, you can replace it. No fear of technical debt.

**The Test**: Could a new developer implement this interface without seeing the current implementation?

**Example**:

```typescript
// Interface is all you need
interface EmailService {
  send(to: string, subject: string, body: string): Promise<void>;
}

// Implementation 1: SendGrid
class SendGridEmailService implements EmailService { /* ... */ }

// Implementation 2: Mailgun (completely different internally)
class MailgunEmailService implements EmailService { /* ... */ }

// Implementation 3: Mock (for testing)
class MockEmailService implements EmailService { /* ... */ }

// Your code doesn't care which implementation
class UserRegistrar {
  constructor(private emailService: EmailService) {}

  async register(email: string) {
    await this.emailService.send(email, "Welcome!", "Thanks for signing up");
  }
}
```

**Benefits**:
- No fear of legacy code (can always rewrite)
- Easy A/B testing (swap implementations)
- Gradual migration (one module at a time)
- Technology independence (not locked in)

---

### 4. Single Responsibility Modules

**Concept**: One module = one person should be able to build and maintain it

**Why It Matters**: Cognitive load is the real bottleneck, not algorithmic complexity

**The Test**: Can you describe the module's purpose in one sentence?

**Examples**:

**Good (Single Responsibility)**:
- `EmailValidator` - Validates email addresses
- `PasswordHasher` - Hashes and verifies passwords
- `UserRepository` - Manages user data persistence

**Bad (Multiple Responsibilities)**:
- `UserManager` - Handles auth, profiles, notifications, permissions, sessions, etc.

**Benefits**:
- One person owns each module
- Easy to understand
- Easy to test
- Easy to replace
- Changes are localized

**Rule of Thumb**: If you can't find a simple name, the module does too much.

---

### 5. Wrap External Dependencies

**Concept**: Never use external libraries directly. Always wrap them behind your own interface.

**Why It Matters**: External code changes. Your interface doesn't.

**Example**:

**Bad (Direct Dependency)**:
```typescript
import { S3 } from 'aws-sdk';

class ProductService {
  uploadImage(file: Buffer) {
    const s3 = new S3();  // ❌ Locked into AWS SDK
    return s3.upload({ /* ... */ }).promise();
  }
}
```

**Good (Wrapped)**:
```typescript
// Your interface (doesn't mention AWS)
interface FileStorage {
  upload(key: string, data: Buffer): Promise<void>;
  download(key: string): Promise<Buffer>;
}

// AWS implementation
class S3FileStorage implements FileStorage {
  private s3 = new S3();

  async upload(key: string, data: Buffer) {
    await this.s3.upload({ /* ... */ }).promise();
  }
}

// Your code uses the interface
class ProductService {
  constructor(private storage: FileStorage) {}

  uploadImage(file: Buffer) {
    return this.storage.upload('products/image.jpg', file);
  }
}
```

**Benefits**:
- Easy to swap vendors (S3 → Google Cloud)
- Easy to test (use MockFileStorage)
- Easy to migrate (add new implementation, switch gradually)
- Insulates from breaking changes

---

## Secondary Principles

### Constant Velocity

**Goal**: Write code at the same speed in year 5 as year 1

**How**:
- Small, focused modules
- Clear interfaces
- No cascading changes
- Easy to add features (new modules)

### Design for Implementability

**Interfaces should be**:
- Simple to implement (< 100 lines)
- One obvious way to use
- Minimal parameters
- Clear error handling

### Optimize for Humans

**Priority**:
1. Human understanding
2. Maintainability
3. Debuggability
4. Performance (only when measured)

### The "One Person Rule"

Each module should be:
- Understandable by one person
- Maintainable by one person
- Testable by one person
- Replaceable by one person

---

## Anti-Patterns to Avoid

### ❌ Leaky Abstractions
Exposing implementation details in interfaces

### ❌ God Objects
Modules that do everything

### ❌ Tight Coupling
Modules that know about each other's internals

### ❌ Framework Lock-In
Architecture depends on specific framework

### ❌ Premature Optimization
Complex design for simple problems

### ❌ Clever Code
Optimizing for brevity over clarity

### ❌ Shared Mutable State
Global state across modules

---

## The Risk Framework

**Problem**: "A lot of small failures = a big failure"

**Risks**:
- Platform changes (OS, cloud provider)
- Technology obsolescence (framework, language)
- Team turnover (key developers leave)
- Requirement evolution (needs change)
- Scale challenges (success creates problems)

**Mitigation**:
- Wrap platform dependencies
- Minimize framework coupling
- Simple, documented modules
- General primitives
- Modular scalability

---

## Real-World Application

### Video Game Engine (Eskil's Example)

**Primitives**: Polygons, textures, vertices
**Modules**: Rendering, physics, networking, input
**Black Boxes**: Each system has clean interface
**Result**: Can rewrite rendering engine without touching gameplay

### E-Commerce Platform

**Primitives**: Product, Order, Payment, User
**Modules**: Catalog, Cart, Checkout, Inventory
**Black Boxes**: Can swap payment providers, database, frontend
**Result**: Add features without slowing down

### Healthcare System

**Primitives**: Patient, Appointment, Prescription, Record
**Modules**: Scheduling, Billing, EHR, Notifications
**Black Boxes**: Comply with regulations, swap vendors
**Result**: Maintain HIPAA compliance while evolving

---

## The Philosophy in Practice

### When Starting a New Project

1. Identify primitives (core data types)
2. Design module boundaries (black boxes)
3. Define interfaces (what, not how)
4. Build one module at a time
5. Test interfaces thoroughly

### When Refactoring Existing Code

1. Find current primitives
2. Identify coupling issues
3. Design better boundaries
4. Extract one module at a time
5. Keep tests passing

### When Adding Features

1. Use existing primitives if possible
2. Add new module (don't modify existing)
3. Connect through interfaces
4. Test in isolation
5. Integration test

---

## Measuring Success

### Developer Velocity
- Time to add feature (constant?)
- Time to fix bug (localized?)
- Time to onboard developer (< 1 day?)

### Code Quality
- Lines per module (< 500?)
- Methods per interface (< 10?)
- Dependencies per module (< 5?)
- Test coverage (> 80%?)

### Maintainability
- Can one person own each module?
- Can modules be rewritten in < 1 week?
- Are changes localized to single modules?

---

## Further Learning

- **Watch**: [Eskil Steenberg's Full Lecture](https://www.youtube.com/watch?v=sSpULGNHyoI) (1 hour)
- **Read**: This repository's [examples](EXAMPLES.md)
- **Practice**: Use the prompts in [usage guide](USAGE.md)

---

**Remember**: Good architecture makes complex systems feel simple, not the other way around.
