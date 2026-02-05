# Usage Guide

Learn how to effectively use Black Box Architecture prompts to refactor code, design systems, and debug issues.

Agents are the recommended daily workflow. Commands/skills are legacy prompts kept for compatibility.

## Table of Contents

- [Recommended Workflow (Agents)](#recommended-workflow-agents)
- [Legacy Quick Start Workflows (Commands)](#legacy-quick-start-workflows-commands)
- [Refactoring Code](#refactoring-code)
- [Planning Architecture](#planning-architecture)
- [Debugging Issues](#debugging-issues)
- [Best Practices](#best-practices)
- [Common Patterns](#common-patterns)

---

## Recommended Workflow (Agents)

Use the orchestrator for most work:

```bash
Ask arch-orchestrator: Analyze UserService and propose black-box boundaries
Ask arch-orchestrator: Design a module map for payments with Stripe + PayPal
Ask arch-orchestrator: Isolate failing integration between PaymentService and OrderService
```

Default to concise output and request a full report only for major changes.

---

## Legacy Quick Start Workflows (Commands)

### Workflow 1: Refactoring Existing Code

```bash
# 1. Navigate to your project
cd your-project/

# 2. Use the refactor command
/arch Analyze the UserService class in src/services/user.ts and refactor it into black box modules
```

**What happens:**
1. AI discovers your code structure
2. Identifies primitives and coupling issues
3. Proposes modular design
4. Provides step-by-step refactoring plan

### Workflow 2: Designing New Features

```bash
# Use the planning command
/arch-plan I need to add a payment processing system to my e-commerce app. It should support Stripe, PayPal, and future payment providers.
```

**What happens:**
1. AI clarifies requirements
2. Identifies system primitives (Payment, PaymentMethod, etc.)
3. Designs black box module boundaries
4. Creates implementation roadmap

### Workflow 3: Debugging Integration Issues

```bash
# Use the debug command
/arch-debug The integration tests between PaymentService and OrderService are failing. Error: "Order not found" when payment completes.
```

**What happens:**
1. AI identifies module boundaries involved
2. Tests interfaces independently
3. Isolates the root cause
4. Proposes fix maintaining black box properties

---

## Refactoring Code

Legacy command examples use `/arch`. You can replace these with `arch-orchestrator` requests.

### Use Case: Breaking Apart a God Object

**Problem**: UserService has grown to 500+ lines, handling authentication, profile management, notifications, and more.

**Solution**:

```bash
/arch The UserService class is too large. Break it into focused, replaceable modules.
```

**Expected Output**:

```markdown
## 🔍 Current Architecture

### Primitives Identified
- **User** (src/models/user.ts:12): Core user data
- **Credential** (src/services/user.ts:45): Auth data
- **Profile** (src/services/user.ts:123): User profile info

### Coupling Issues
- ❌ UserService directly calls EmailService (src/services/user.ts:234)
- ❌ Authentication logic mixed with profile updates (src/services/user.ts:156-289)

## 🎯 Proposed Black Box Design

### Module: AuthenticationService
**Responsibility**: Handle user login, logout, and credential verification
**Interface**:
```typescript
interface AuthenticationService {
  authenticate(email: string, password: string): AuthResult;
  logout(userId: string): void;
}
```

### Module: ProfileService
**Responsibility**: Manage user profile data
[...]

## 📝 Implementation Steps
1. Extract AuthenticationService (2 hours)
2. Extract ProfileService (1.5 hours)
[...]
```

### Use Case: Wrapping External Dependencies

**Problem**: Direct use of AWS SDK throughout codebase makes testing hard and locks into AWS.

**Solution**:

```bash
/arch We're using AWS SDK directly in 15 files. Create a black box storage abstraction.
```

**Expected Output**:

```markdown
## 🎯 Proposed Black Box Design

### Module: StorageService
**Responsibility**: Provide platform-agnostic file storage
**Interface**:
```typescript
interface StorageService {
  upload(key: string, data: Buffer): Promise<void>;
  download(key: string): Promise<Buffer>;
  delete(key: string): Promise<void>;
}
```

**Implementations**:
- S3StorageService (wraps AWS SDK)
- LocalStorageService (for development)
- MockStorageService (for testing)
```

---

## Planning Architecture

Legacy command examples use `/arch-plan`. You can replace these with `arch-orchestrator` requests.

### Use Case: Designing a New System

**Scenario**: Building a real-time chat application

**Prompt**:

```bash
/arch-plan Design architecture for a real-time chat app with:
- Multiple message types (text, images, files)
- User presence indicators
- Message history persistence
- Support for web and mobile clients
```

**Expected Output**:

```markdown
## System Primitives

### Primitive: Message
**Definition**: A single unit of communication between users
**Properties**: id, sender, recipient, content, type, timestamp
**Operations**: send, receive, delete, edit
**Why Primitive**: All features revolve around passing messages

### Primitive: User
**Definition**: An authenticated participant
**Properties**: id, username, status, lastSeen
**Operations**: login, logout, updateStatus
**Why Primitive**: Messages require sender/recipient

## Module Architecture

### Module: MessageStore
**Responsibility**: Persist and retrieve message history
**Interface**:
```typescript
interface MessageStore {
  save(message: Message): Promise<void>;
  getHistory(userId: string, limit: number): Promise<Message[]>;
}
```
**Dependencies**: None (black box storage)
**Replaceability**: Can swap MongoDB for PostgreSQL

### Module: PresenceTracker
**Responsibility**: Track user online/offline status
**Interface**: [...]

## Implementation Roadmap

### Phase 1: Foundation (Week 1)
- Build Message primitive
- Implement MessageStore with in-memory backend
- Create simple test client

### Phase 2: Real-time Layer (Week 2)
- Add WebSocket handling
- Implement PresenceTracker
- Test with web client

[...]

## Risk Assessment
| Risk | Mitigation |
|------|------------|
| WebSocket scalability | Abstract connection handling, can swap Socket.io for custom solution |
```

---

## Debugging Issues

Legacy command examples use `/arch-debug`. You can replace these with `arch-orchestrator` requests.

### Use Case: Integration Test Failures

**Problem**: Tests pass individually but fail when modules interact

**Prompt**:

```bash
/arch-debug Integration test failing: PaymentService returns success but OrderService doesn't receive the order_id. Unit tests pass for both.
```

**Expected Output**:

```markdown
## 🐛 Bug Analysis

**Expected**: OrderService receives order_id after payment
**Actual**: order_id is undefined in OrderService
**Location**: Integration point between modules

## 🔬 Module Boundary Analysis

### Module: PaymentService
**Interface**:
```typescript
interface PaymentService {
  processPayment(orderId: string, amount: number): Promise<PaymentResult>;
}
```

### Module: OrderService
**Interface**:
```typescript
interface OrderService {
  confirmPayment(orderId: string, paymentId: string): Promise<void>;
}
```

## 🧪 Isolation Tests

Test 1: PaymentService produces correct output
```typescript
test("payment service returns orderId", () => {
  const result = await paymentService.processPayment("order-123", 100);
  expect(result.orderId).toBe("order-123"); // ✅ PASSES
});
```

Test 2: OrderService accepts correct input
```typescript
test("order service accepts orderId", () => {
  await orderService.confirmPayment("order-123", "payment-456");
  // ✅ PASSES
});
```

## 🎯 Root Cause

**Issue**: Interface mismatch - PaymentResult has `order_id` (snake_case) but OrderService expects `orderId` (camelCase)

**Module Violation**: Leaky abstraction - internal data format exposed in interface

## 🔧 Proposed Fix

Standardize interface to use consistent naming:

```typescript
interface PaymentResult {
  orderId: string;  // Changed from order_id
  paymentId: string;
  success: boolean;
}
```
```

---

## Best Practices

### 1. Start Small

Don't refactor the entire codebase at once.

**Good**:
```bash
/arch Refactor the UserService authentication methods into a separate module
```

**Bad**:
```bash
/arch Refactor the entire src/ directory
```

### 2. Provide Context

Help the AI understand your codebase:

**Good**:
```bash
/arch The PaymentService in src/services/payment/ handles Stripe integration. It's grown to 800 lines. Break it into replaceable payment provider modules.
```

**Better**:
```bash
/arch The PaymentService handles Stripe integration but we need to add PayPal and Apple Pay. Refactor to support multiple payment providers through a black box interface.
```

### 3. Specify Your Language/Framework

```bash
/arch-plan Design a REST API architecture using Go and PostgreSQL for a inventory management system
```

### 4. Use Iterative Refinement

```bash
# First pass
/arch Refactor UserService

# Review the proposal, then refine
/arch The proposed AuthenticationService looks good, but can you make the password hashing strategy pluggable?
```

### 5. Ask for Specific Output

```bash
/arch Analyze the database access layer and show me before/after code examples for wrapping the SQL queries
```

---

## Common Patterns

### Pattern 1: Repository Pattern

**When**: Separating data access from business logic

```bash
/arch Create a repository pattern for our User data access, currently using Mongoose directly in 20 files
```

### Pattern 2: Adapter Pattern

**When**: Integrating with external systems

```bash
/arch We need to support multiple email providers (SendGrid, Mailgun, AWS SES). Design an email service abstraction.
```

### Pattern 3: Plugin Architecture

**When**: Building extensible systems

```bash
/arch-plan Design a plugin system for our reporting tool that allows third-party report types
```

### Pattern 4: Strangler Fig (Legacy Refactoring)

**When**: Gradually replacing legacy code

```bash
/arch We have a legacy PHP monolith. Design a strategy to extract the user authentication module into a separate Node.js service without breaking existing functionality.
```

---

## Real-World Scenarios

### Scenario: Monolith to Microservices

```bash
# Phase 1: Identify boundaries
/arch Analyze our monolith in src/ and identify potential service boundaries

# Phase 2: Extract one service
/arch Extract the notification system (email, SMS, push) into a standalone module with a clean interface

# Phase 3: Plan deployment
/arch-plan How should we deploy the NotificationService separately while maintaining the existing monolith?
```

### Scenario: Performance Optimization

```bash
# Identify bottlenecks
/arch-debug Our API response time increased from 100ms to 2s. The issue is in the product search endpoint.

# Design optimized module
/arch Redesign the ProductSearchService to support caching without changing the public interface
```

### Scenario: Adding New Features

```bash
# Plan the feature
/arch-plan Add OAuth social login (Google, GitHub, Facebook) to our existing email/password authentication

# Implement incrementally
/arch Implement the OAuth abstraction layer first, then we'll add providers one by one
```

---

## Tips for Success

### ✅ Do:
- Start with one module at a time
- Ask for before/after code examples
- Validate the proposed design before implementing
- Use the AI's structured output format
- Test after each refactoring step

### ❌ Don't:
- Try to refactor everything at once
- Skip the design phase and jump to implementation
- Ignore the risk assessment
- Break working code without tests
- Forget to commit incrementally

---

## Next Steps

- See [Examples](EXAMPLES.md) for complete code transformations
- Read [Principles](PRINCIPLES.md) to understand the methodology
- Check [Contributing](CONTRIBUTING.md) to share your patterns
