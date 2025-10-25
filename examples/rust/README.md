# Rust Black Box Examples

Trait-based architecture for modular, safe Rust code.

## Examples

- [Trait Objects](trait-objects/) - Dynamic dispatch for replaceability
- [Generic Traits](generic-traits/) - Static dispatch with generics
- [Plugin System](plugin-system/) - Loadable modules

## Key Patterns

### 1. Traits as Interfaces

```rust
pub trait Storage {
    fn write(&self, key: &str, data: &[u8]) -> Result<()>;
    fn read(&self, key: &str) -> Result<Vec<u8>>;
}

pub struct FileStorage {
    base_path: PathBuf,
}

impl Storage for FileStorage {
    fn write(&self, key: &str, data: &[u8]) -> Result<()> {
        // Implementation
    }
}
```

### 2. Generic Dependency Injection

```rust
pub struct ConfigManager<S: Storage> {
    storage: S,
}

impl<S: Storage> ConfigManager<S> {
    pub fn new(storage: S) -> Self {
        ConfigManager { storage }
    }
}
```
