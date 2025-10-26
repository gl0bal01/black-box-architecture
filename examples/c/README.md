# C Black Box Examples

Black box architecture in C using function pointers and opaque types - the foundation of Eskil Steenberg's approach.

## Key Patterns

### 1. Opaque Pointers (Information Hiding)

**Before (Exposed Structure)**:
```c
// user.h - Exposes implementation
typedef struct {
    char name[256];
    char email[256];
    int age;
    // Everyone can see and access internals
} User;

void user_create(User* user, const char* name);
```

**After (Opaque Type)**:
```c
// user.h - Black box interface
typedef struct User User;  // Forward declaration only

// Public API
User* user_create(const char* name, const char* email);
void user_destroy(User* user);
const char* user_get_name(User* user);
void user_set_age(User* user, int age);
```

```c
// user.c - Implementation hidden
struct User {
    char* name;
    char* email;
    int age;
    // Implementation details hidden
};

User* user_create(const char* name, const char* email) {
    User* user = malloc(sizeof(User));
    user->name = strdup(name);
    user->email = strdup(email);
    user->age = 0;
    return user;
}

void user_destroy(User* user) {
    free(user->name);
    free(user->email);
    free(user);
}
```

**Benefits**:
- ✅ Can change internal structure without breaking code
- ✅ Prevents direct field access
- ✅ Encapsulation in C

---

### 2. Function Pointers (Interface Pattern)

**Before (Tight Coupling)**:
```c
void process_order(Order* order) {
    // Hard-coded file storage
    FILE* f = fopen("orders.dat", "ab");
    fwrite(order, sizeof(Order), 1, f);
    fclose(f);
}
```

**After (Black Box Storage Interface)**:
```c
// storage.h - Interface definition
typedef struct Storage Storage;

typedef struct StorageVTable {
    int (*save)(Storage* self, const char* key, const void* data, size_t size);
    int (*load)(Storage* self, const char* key, void* data, size_t size);
    void (*destroy)(Storage* self);
} StorageVTable;

struct Storage {
    void* impl;                    // Implementation-specific data
    const StorageVTable* vtable;   // Function pointers
};

// Helper macros for clean API
#define storage_save(s, k, d, sz) ((s)->vtable->save((s), (k), (d), (sz)))
#define storage_load(s, k, d, sz) ((s)->vtable->load((s), (k), (d), (sz)))
#define storage_destroy(s) ((s)->vtable->destroy(s))
```

```c
// file_storage.c - File implementation
typedef struct {
    char* base_path;
} FileStorageImpl;

static int file_storage_save(Storage* self, const char* key,
                             const void* data, size_t size) {
    FileStorageImpl* impl = (FileStorageImpl*)self->impl;
    // File implementation
    char path[512];
    snprintf(path, sizeof(path), "%s/%s", impl->base_path, key);
    FILE* f = fopen(path, "wb");
    if (!f) return -1;
    fwrite(data, size, 1, f);
    fclose(f);
    return 0;
}

static int file_storage_load(Storage* self, const char* key,
                             void* data, size_t size) {
    // Implementation
    return 0;
}

static void file_storage_destroy(Storage* self) {
    FileStorageImpl* impl = (FileStorageImpl*)self->impl;
    free(impl->base_path);
    free(impl);
    free(self);
}

static const StorageVTable file_storage_vtable = {
    .save = file_storage_save,
    .load = file_storage_load,
    .destroy = file_storage_destroy
};

Storage* file_storage_create(const char* base_path) {
    Storage* storage = malloc(sizeof(Storage));
    FileStorageImpl* impl = malloc(sizeof(FileStorageImpl));
    impl->base_path = strdup(base_path);
    storage->impl = impl;
    storage->vtable = &file_storage_vtable;
    return storage;
}
```

```c
// memory_storage.c - In-memory implementation
typedef struct {
    void* buffer;
    size_t capacity;
} MemoryStorageImpl;

static int memory_storage_save(Storage* self, const char* key,
                               const void* data, size_t size) {
    // Memory implementation
    return 0;
}

static const StorageVTable memory_storage_vtable = {
    .save = memory_storage_save,
    .load = memory_storage_load,
    .destroy = memory_storage_destroy
};

Storage* memory_storage_create(size_t capacity) {
    Storage* storage = malloc(sizeof(Storage));
    MemoryStorageImpl* impl = malloc(sizeof(MemoryStorageImpl));
    impl->buffer = malloc(capacity);
    impl->capacity = capacity;
    storage->impl = impl;
    storage->vtable = &memory_storage_vtable;
    return storage;
}
```

```c
// Usage - Completely replaceable
void process_order(Order* order, Storage* storage) {
    // Works with ANY storage implementation
    storage_save(storage, "order.dat", order, sizeof(Order));
}

int main() {
    // Can use file storage
    Storage* file_store = file_storage_create("/data");
    process_order(&order, file_store);
    storage_destroy(file_store);

    // Or memory storage
    Storage* mem_store = memory_storage_create(1024);
    process_order(&order, mem_store);
    storage_destroy(mem_store);

    return 0;
}
```

**Benefits**:
- ✅ Can swap implementations at runtime
- ✅ Easy to test (mock storage)
- ✅ No recompilation needed for new implementations
- ✅ True polymorphism in C

---

### 3. Module Pattern (Eskil's Approach)

**module.h - Public Interface**:
```c
#ifndef MODULE_H
#define MODULE_H

// Forward declaration - opaque type
typedef struct Module Module;

// Module lifecycle
Module* module_create(const char* name);
void module_destroy(Module* module);

// Public operations
void module_process(Module* module, const void* data);
int module_get_status(const Module* module);

#endif // MODULE_H
```

**module.c - Hidden Implementation**:
```c
#include "module.h"
#include <stdlib.h>
#include <string.h>

// Structure definition hidden in .c file
struct Module {
    char* name;
    int status;
    void* private_data;  // Implementation details
    // Only visible in this translation unit
};

Module* module_create(const char* name) {
    Module* m = malloc(sizeof(Module));
    m->name = strdup(name);
    m->status = 0;
    m->private_data = NULL;
    return m;
}

void module_destroy(Module* module) {
    if (!module) return;
    free(module->name);
    free(module->private_data);
    free(module);
}

void module_process(Module* module, const void* data) {
    // Implementation details
    module->status = 1;
}

int module_get_status(const Module* module) {
    return module ? module->status : -1;
}
```

**Benefits**:
- ✅ True encapsulation (struct only in .c file)
- ✅ Can change internals without recompiling users
- ✅ Clear API surface
- ✅ Unix philosophy (small, focused modules)

---

## Real-World Example: Graphics Engine

**Before (Monolithic)**:
```c
void render_scene() {
    // Everything coupled together
    glClear(GL_COLOR_BUFFER_BIT);
    glBegin(GL_TRIANGLES);
    // Direct OpenGL calls everywhere
    glVertex3f(0.0f, 1.0f, 0.0f);
    glEnd();
    glFlush();
}
```

**After (Black Box Renderer)**:
```c
// renderer.h - Platform-agnostic interface
typedef struct Renderer Renderer;

Renderer* renderer_create(void);
void renderer_destroy(Renderer* r);
void renderer_clear(Renderer* r);
void renderer_draw_triangle(Renderer* r, float x, float y, float z);
void renderer_present(Renderer* r);
```

```c
// opengl_renderer.c - OpenGL implementation
Renderer* opengl_renderer_create(void) {
    // OpenGL-specific initialization
}

void opengl_renderer_clear(Renderer* r) {
    glClear(GL_COLOR_BUFFER_BIT);
}
```

```c
// vulkan_renderer.c - Vulkan implementation (completely different internally)
Renderer* vulkan_renderer_create(void) {
    // Vulkan-specific initialization
}

void vulkan_renderer_clear(Renderer* r) {
    // Vulkan clear implementation
}
```

```c
// Usage - Renderer is replaceable
int main() {
    #ifdef USE_OPENGL
    Renderer* r = opengl_renderer_create();
    #else
    Renderer* r = vulkan_renderer_create();
    #endif

    renderer_clear(r);
    renderer_draw_triangle(r, 0.0f, 1.0f, 0.0f);
    renderer_present(r);
    renderer_destroy(r);

    return 0;
}
```

---

## Testing with Black Boxes

```c
// test_storage.c - Test ANY storage implementation
void test_storage_interface(Storage* storage) {
    // Test works for file, memory, or any other implementation
    const char* test_data = "hello world";

    // Test save
    int result = storage_save(storage, "test", test_data, strlen(test_data));
    assert(result == 0);

    // Test load
    char buffer[256] = {0};
    result = storage_load(storage, "test", buffer, sizeof(buffer));
    assert(result == 0);
    assert(strcmp(buffer, test_data) == 0);
}

int main() {
    // Test file storage
    Storage* file = file_storage_create("/tmp");
    test_storage_interface(file);
    storage_destroy(file);

    // Test memory storage (same test!)
    Storage* memory = memory_storage_create(1024);
    test_storage_interface(memory);
    storage_destroy(memory);

    printf("All tests passed!\n");
    return 0;
}
```

---

## Eskil's Principles in C

From his graphics engines and networked games:

1. **Opaque types** - Hide structure definitions
2. **Function pointers** - Create interfaces/vtables
3. **One header, one source** - Clear module boundaries
4. **Allocate in constructor** - `module_create()` returns pointer
5. **Free in destructor** - `module_destroy()` cleans up
6. **Const correctness** - Mark read-only params
7. **Null checks** - Defensive programming

## Common Patterns

### Pattern 1: Context Object
```c
typedef struct Context Context;
Context* context_create(void);
void context_set_option(Context* ctx, const char* key, const char* value);
void context_destroy(Context* ctx);
```

### Pattern 2: Callback Registration
```c
typedef void (*EventCallback)(void* user_data, const Event* event);
void module_register_callback(Module* m, EventCallback cb, void* user_data);
```

### Pattern 3: Error Handling
```c
typedef enum {
    STATUS_OK = 0,
    STATUS_ERROR = -1,
    STATUS_INVALID_ARG = -2
} Status;

Status module_operation(Module* m, const char* data);
```

---

## Building and Running

```bash
# Compile with separate compilation
gcc -c user.c -o user.o
gcc -c main.c -o main.o
gcc user.o main.o -o program

# Can replace user.c without recompiling main.c
gcc -c user_v2.c -o user.o
gcc user.o main.o -o program  # main.o unchanged!
```

## References

- Eskil's C codebase examples
- Unix design philosophy
- X11 and OpenGL APIs (classic black box C)
- GTK+ and GLib (opaque types in practice)

## Contributing

Add more C examples via pull request!
