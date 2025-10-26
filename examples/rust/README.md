# Rust Black Box Examples

Trait-based architecture for modular, safe Rust code.

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
