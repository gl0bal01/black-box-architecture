# Python Black Box Examples

Before/after refactoring examples demonstrating black box architecture in Python.

## Examples

- [Repository Pattern](repository-pattern/) - Separating data access from business logic
- [Service Abstraction](service-abstraction/) - Wrapping external APIs
- [Plugin Architecture](plugin-architecture/) - Extensible module system

## Running Examples

```bash
cd python/repository-pattern
python before.py  # See the coupled version
python after.py   # See the black box version
python tests.py   # Run tests showing replaceability
```

## Key Patterns

### 1. Abstract Base Classes for Interfaces

```python
from abc import ABC, abstractmethod

class Storage(ABC):
    @abstractmethod
    def save(self, key: str, data: bytes) -> None:
        pass
```

### 2. Dependency Injection

```python
class UserService:
    def __init__(self, repository: UserRepository):
        self.repo = repository  # Injected, not created
```

### 3. Protocol (Python 3.8+)

```python
from typing import Protocol

class Authenticator(Protocol):
    def authenticate(self, username: str, password: str) -> bool:
        ...
```

## Contributing

Add more examples via pull request!
