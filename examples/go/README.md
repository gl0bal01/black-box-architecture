# Go Black Box Examples

Interface-driven design patterns in Go.

## Examples

- [Interface Composition](interface-composition/) - Combining small interfaces
- [Dependency Injection](dependency-injection/) - Using interfaces for testability
- [Repository Pattern](repository-pattern/) - Data access abstraction

## Key Patterns

### 1. Small, Focused Interfaces

```go
type Logger interface {
    Info(message string)
    Error(message string)
}

type Storage interface {
    Save(key string, data []byte) error
    Load(key string) ([]byte, error)
}
```

### 2. Struct Composition

```go
type OrderService struct {
    logger  Logger
    storage Storage
    mailer  Mailer
}

func NewOrderService(l Logger, s Storage, m Mailer) *OrderService {
    return &OrderService{logger: l, storage: s, mailer: m}
}
```
