# TypeScript Black Box Examples

Before/after examples showing modular, testable TypeScript architecture.

## Examples

- [Interface Segregation](interface-segregation/) - Clean module boundaries
- [Dependency Injection](dependency-injection/) - Testable, swappable dependencies
- [Adapter Pattern](adapter-pattern/) - Wrapping external libraries

## Key Patterns

### 1. Interface-First Design

```typescript
interface EmailService {
  send(to: string, subject: string, body: string): Promise<void>;
}

class SendGridEmail implements EmailService {
  async send(to: string, subject: string, body: string) {
    // Implementation
  }
}
```

### 2. Constructor Injection

```typescript
class UserRegistrar {
  constructor(
    private emailService: EmailService,
    private userRepo: UserRepository
  ) {}
}
```

### 3. Factory Pattern

```typescript
function createEmailService(config: Config): EmailService {
  return config.provider === 'sendgrid'
    ? new SendGridEmail(config)
    : new MailgunEmail(config);
}
```
