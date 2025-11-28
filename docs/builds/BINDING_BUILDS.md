# GBLN Language Bindings - Build & Test Guide

**Version:** 2.0  
**Last Updated:** 2025-11-28  
**Status:** Production - VERIFIED & TESTED

---

## Overview

GBLN provides bindings for 12+ languages, all using the pre-built C FFI libraries as foundation:

```
Rust Core → C FFI (cross builds) → Pre-built libs (Git) → Language Bindings → User API
```

**All bindings share:**
- Same C FFI libraries from `core/ffi/libs/` (committed to Git)
- Same platform support (8 platforms)
- Same quality requirements (tested on all platforms)
- Same documentation patterns

---

## Pre-Built Libraries (CRITICAL)

**ALL language bindings use pre-built C FFI libraries from `core/ffi/libs/`.**

**DO NOT build C FFI as part of binding development!**

### Library Locations

```
core/ffi/libs/
├── linux-x64/libgbln.so          # 573K
├── linux-arm64/libgbln.so        # 583K
├── freebsd-x64/libgbln.so        # 548K
├── freebsd-arm64/libgbln.so      # 639K
├── windows-x64/gbln.dll          # 1.5M
├── android-arm64/libgbln.so      # 627K
├── android-x64/libgbln.so        # 617K
└── macos-arm64/libgbln.dylib     # 553K
```

**These are:**
- ✅ Committed to Git
- ✅ Built with `cross` tool (verified working)
- ✅ Used by ALL language bindings
- ✅ Included in distributed packages (pip, npm, gem, etc.)

**DO NOT:**
- ❌ Rebuild C FFI during binding development
- ❌ Use `target/` directory libraries (use `libs/` instead)
- ❌ Build C FFI on user machines
- ❌ Assume users have Rust/Docker installed

---

## Reference Implementation

**Ticket #100 (Python bindings)** is the reference implementation for ALL other bindings.

**Before starting any binding ticket (#101-111):**

1. ✅ Read Ticket #100 completely
2. ✅ Study `bindings/python/` implementation
3. ✅ Understand library loading from `core/ffi/libs/`
4. ✅ Copy library selection logic (platform detection)
5. ✅ Copy `bindings/TEMPLATE/` as starting point

---

## Platform Matrix

**Every binding MUST support all 8 platforms.**

| Platform | Architecture | Library | Size | Status |
|----------|-------------|---------|------|--------|
| Linux | x86_64 | libgbln.so | 573K | ✅ |
| Linux | ARM64 | libgbln.so | 583K | ✅ |
| FreeBSD | x86_64 | libgbln.so | 548K | ✅ |
| FreeBSD | ARM64 | libgbln.so | 639K | ✅ |
| Windows | x86_64 | gbln.dll | 1.5M | ✅ |
| Android | ARM64 | libgbln.so | 627K | ✅ |
| Android | x86_64 | libgbln.so | 617K | ✅ |
| macOS | ARM64 | libgbln.dylib | 553K | ✅ |

---

## Library Loading Pattern

**All bindings MUST implement platform detection and library loading.**

### Python Example (Reference)

```python
import ctypes
import platform
import sys
import os
from pathlib import Path

def _load_library():
    """Load platform-specific GBLN library from core/ffi/libs/"""
    
    system = platform.system()
    machine = platform.machine()
    
    # Map platform to library directory
    if system == "Linux":
        if machine in ("x86_64", "AMD64"):
            lib_dir = "linux-x64"
            lib_name = "libgbln.so"
        elif machine in ("aarch64", "arm64"):
            lib_dir = "linux-arm64"
            lib_name = "libgbln.so"
        else:
            raise RuntimeError(f"Unsupported Linux architecture: {machine}")
    
    elif system == "Darwin":  # macOS
        if machine == "arm64":
            lib_dir = "macos-arm64"
            lib_name = "libgbln.dylib"
        else:
            raise RuntimeError(f"Unsupported macOS architecture: {machine}")
    
    elif system == "Windows":
        if machine in ("AMD64", "x86_64"):
            lib_dir = "windows-x64"
            lib_name = "gbln.dll"
        else:
            raise RuntimeError(f"Unsupported Windows architecture: {machine}")
    
    elif system == "FreeBSD":
        if machine == "amd64":
            lib_dir = "freebsd-x64"
            lib_name = "libgbln.so"
        elif machine == "arm64":
            lib_dir = "freebsd-arm64"
            lib_name = "libgbln.so"
        else:
            raise RuntimeError(f"Unsupported FreeBSD architecture: {machine}")
    
    elif system == "Android":
        if machine in ("aarch64", "arm64"):
            lib_dir = "android-arm64"
            lib_name = "libgbln.so"
        elif machine in ("x86_64", "AMD64"):
            lib_dir = "android-x64"
            lib_name = "libgbln.so"
        else:
            raise RuntimeError(f"Unsupported Android architecture: {machine}")
    
    else:
        raise RuntimeError(f"Unsupported platform: {system}")
    
    # Construct path to library
    # From bindings/python/ to core/ffi/libs/{platform}/
    binding_dir = Path(__file__).parent
    lib_path = binding_dir / "../../core/ffi/libs" / lib_dir / lib_name
    lib_path = lib_path.resolve()
    
    if not lib_path.exists():
        raise FileNotFoundError(
            f"GBLN library not found: {lib_path}\n"
            f"Platform: {system} {machine}\n"
            f"Expected: {lib_dir}/{lib_name}"
        )
    
    return ctypes.CDLL(str(lib_path))

# Load library on module import
_lib = _load_library()
```

### Adapt for Each Language

**JavaScript (Node.js):**
```javascript
const ffi = require('ffi-napi');
const path = require('path');
const os = require('os');

function loadLibrary() {
    const platform = os.platform();
    const arch = os.arch();
    
    let libDir, libName;
    
    if (platform === 'linux') {
        if (arch === 'x64') {
            libDir = 'linux-x64';
            libName = 'libgbln.so';
        } else if (arch === 'arm64') {
            libDir = 'linux-arm64';
            libName = 'libgbln.so';
        }
    } else if (platform === 'darwin') {
        libDir = 'macos-arm64';
        libName = 'libgbln.dylib';
    } else if (platform === 'win32') {
        libDir = 'windows-x64';
        libName = 'gbln.dll';
    }
    
    const libPath = path.join(__dirname, '../../core/ffi/libs', libDir, libName);
    return ffi.Library(libPath, { /* FFI definitions */ });
}
```

**Go:**
```go
package gbln

/*
#cgo linux,amd64 LDFLAGS: -L../../core/ffi/libs/linux-x64 -lgbln
#cgo linux,arm64 LDFLAGS: -L../../core/ffi/libs/linux-arm64 -lgbln
#cgo darwin,arm64 LDFLAGS: -L../../core/ffi/libs/macos-arm64 -lgbln
#cgo windows,amd64 LDFLAGS: -L../../core/ffi/libs/windows-x64 -lgbln
#cgo freebsd,amd64 LDFLAGS: -L../../core/ffi/libs/freebsd-x64 -lgbln
#cgo freebsd,arm64 LDFLAGS: -L../../core/ffi/libs/freebsd-arm64 -lgbln

#include <stdlib.h>
*/
import "C"
```

---

## Package Distribution

**All bindings MUST include pre-built libraries in distributed packages.**

### Python (wheel)

```python
# setup.py
from setuptools import setup

setup(
    name="gbln",
    version="0.1.0",
    packages=["gbln"],
    package_data={
        "gbln": [
            "../../core/ffi/libs/linux-x64/*.so",
            "../../core/ffi/libs/linux-arm64/*.so",
            "../../core/ffi/libs/freebsd-x64/*.so",
            "../../core/ffi/libs/freebsd-arm64/*.so",
            "../../core/ffi/libs/windows-x64/*.dll",
            "../../core/ffi/libs/android-arm64/*.so",
            "../../core/ffi/libs/android-x64/*.so",
            "../../core/ffi/libs/macos-arm64/*.dylib",
        ]
    },
)
```

### JavaScript (npm)

```json
{
  "name": "gbln",
  "version": "0.1.0",
  "files": [
    "dist/",
    "../../core/ffi/libs/**/*.so",
    "../../core/ffi/libs/**/*.dll",
    "../../core/ffi/libs/**/*.dylib"
  ]
}
```

### Result

Users install package and get working binaries:
```bash
pip install gbln       # All 8 platform libraries included
npm install gbln       # All 8 platform libraries included
gem install gbln       # All 8 platform libraries included
```

**NO compilation on user machine!**

---

## Testing Requirements

### Minimum Test Coverage

**Every binding MUST have:**

1. **Unit Tests (40%)** - Individual function tests
2. **Integration Tests (40%)** - Parse → Serialize → Parse round-trips
3. **Platform Tests (20%)** - Verify library loading works on each platform

**Minimum:** 100+ test cases

### Test Structure

```
bindings/{language}/tests/
├── test_ffi_loading.{ext}        # Library loading for all platforms
├── test_value_conversion.{ext}   # Type conversion (GBLN ↔ Native)
├── test_parse.{ext}              # Parse GBLN strings
├── test_serialize.{ext}          # Serialize to GBLN
├── test_roundtrip.{ext}          # Full round-trip
├── test_memory.{ext}             # No memory leaks
└── test_errors.{ext}             # Error handling
```

### Platform Testing

**Test on your development machine** (should auto-detect platform):

```bash
# Python
cd bindings/python
pytest -v

# JavaScript
cd bindings/javascript
npm test

# Go
cd bindings/go
go test ./...
```

**The library loader automatically selects the correct platform library.**

### Acceptance Criteria

- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] Platform detection works correctly
- [ ] Correct library loads on each platform
- [ ] No memory leaks (verified with valgrind/leaks)
- [ ] UTF-8 handling correct
- [ ] Error messages helpful

---

## C FFI Interface

**All bindings use the same C FFI functions.**

### Core Functions

```c
// Parse GBLN string
GblnValue* gbln_parse(const char* input, GblnError** error);

// Serialize to GBLN
char* gbln_serialize(const GblnValue* value);

// Free memory
void gbln_value_free(GblnValue* value);
void gbln_string_free(char* str);
void gbln_error_free(GblnError* error);

// Type queries
GblnValueType gbln_value_type(const GblnValue* value);

// Value extraction
int64_t gbln_value_as_i64(const GblnValue* value);
uint64_t gbln_value_as_u64(const GblnValue* value);
double gbln_value_as_f64(const GblnValue* value);
const char* gbln_value_as_string(const GblnValue* value);
bool gbln_value_as_bool(const GblnValue* value);

// Object operations
const char** gbln_object_keys(const GblnValue* value, size_t* count);
GblnValue* gbln_object_get(const GblnValue* value, const char* key);

// Array operations
size_t gbln_array_length(const GblnValue* value);
GblnValue* gbln_array_get(const GblnValue* value, size_t index);
```

**See:** `core/ffi/src/lib.rs` for complete API

---

## Memory Management

**CRITICAL:** All pointers returned by C FFI MUST be freed!

### Rules

1. **`GblnValue*`** → Call `gbln_value_free()` when done
2. **`char*`** (strings) → Call `gbln_string_free()` when done
3. **`GblnError*`** → Call `gbln_error_free()` when done
4. **`char**`** (key arrays) → Call `gbln_keys_free()` when done

### Language-Specific Solutions

**Python:**
```python
import weakref

class Value:
    def __init__(self, ptr):
        self._ptr = ptr
        weakref.finalize(self, _lib.gbln_value_free, ptr)
```

**JavaScript:**
```javascript
const registry = new FinalizationRegistry(ptr => {
    _lib.gbln_value_free(ptr);
});

class Value {
    constructor(ptr) {
        this._ptr = ptr;
        registry.register(this, ptr, this);
    }
}
```

**Go:**
```go
import "runtime"

type Value struct {
    ptr *C.GblnValue
}

func NewValue(ptr *C.GblnValue) *Value {
    v := &Value{ptr: ptr}
    runtime.SetFinalizer(v, func(v *Value) {
        C.gbln_value_free(v.ptr)
    })
    return v
}
```

**C++:**
```cpp
class Value {
    GblnValue* _ptr;
public:
    Value(GblnValue* ptr) : _ptr(ptr) {}
    ~Value() { gbln_value_free(_ptr); }
    
    // Disable copy, enable move
    Value(const Value&) = delete;
    Value& operator=(const Value&) = delete;
    Value(Value&& other) : _ptr(other._ptr) { other._ptr = nullptr; }
};
```

---

## Type Conversion

**GBLN ↔ Native type mapping:**

| GBLN Type | Python | JavaScript | Java | Go | Rust | Swift |
|-----------|--------|------------|------|-----|------|-------|
| I8-I64 | `int` | `number` | `long` | `int64` | `i64` | `Int64` |
| U8-U64 | `int` | `number` | `long` | `uint64` | `u64` | `UInt64` |
| F32, F64 | `float` | `number` | `double` | `float64` | `f64` | `Double` |
| Str | `str` | `string` | `String` | `string` | `String` | `String` |
| Bool | `bool` | `boolean` | `boolean` | `bool` | `bool` | `Bool` |
| Null | `None` | `null` | `null` | `nil` | `None` | `nil` |
| Object | `dict` | `object` | `Map` | `map` | `HashMap` | `Dictionary` |
| Array | `list` | `array` | `List` | `[]T` | `Vec` | `Array` |

---

## Documentation Requirements

### README.md Template

```markdown
# GBLN - {Language} Bindings

{Language} bindings for GBLN (Goblin Bounded Lean Notation).

## Installation

{language_specific_install_command}

## Platform Support

✅ **All 8 platforms supported out-of-the-box:**

- Linux (x86_64, ARM64)
- FreeBSD (x86_64, ARM64)
- Windows (x86_64)
- Android (ARM64, x86_64)
- macOS (ARM64)

Pre-compiled libraries included. No build tools required.

## Quick Start

{language_specific_example}

## API Documentation

{link_to_docs}
```

---

## Quality Checklist

**Before marking binding ticket complete:**

- [ ] All code follows language idioms
- [ ] All comments in BBC English
- [ ] No files >400 lines
- [ ] No generic names (`utils`, `helpers`, `common`)
- [ ] Type annotations/hints where supported
- [ ] Memory management correct (no leaks)
- [ ] All tests pass
- [ ] Platform detection works
- [ ] All 8 libraries load correctly
- [ ] README with platform support info
- [ ] Installation instructions
- [ ] Usage examples
- [ ] API documentation
- [ ] Package includes all libraries
- [ ] Package installable (pip/npm/gem/etc.)

---

## Language-Specific Notes

### Python (#100)
- Use `ctypes.CDLL()` for library loading
- Use `weakref.finalize()` for memory management
- Type hints for all public API

### JavaScript (#101)
- Use `ffi-napi` for FFI or WASM
- Use `FinalizationRegistry` for memory
- TypeScript definitions required

### Swift (#102)
- Use `dlopen()` + Swift/C interop
- Use `deinit` for cleanup
- Native feel with Swift naming conventions

### Kotlin (#103)
- Use JNA for FFI
- Use `try-with-resources` for memory
- Idiomatic Kotlin API

### Go (#104)
- Use CGO with conditional compilation
- Use `runtime.SetFinalizer()` for memory
- Idiomatic Go API

### Others (#105-111)
- Follow language best practices
- Adapt memory management to language
- Provide idiomatic API

---

## Summary

**Building Language Bindings:**

1. ✅ Use pre-built libraries from `core/ffi/libs/` (DO NOT rebuild)
2. ✅ Implement platform detection + library loading
3. ✅ Include all 8 libraries in package distribution
4. ✅ Test on your platform (auto-detects correct library)
5. ✅ Users install package and it just works (no compilation)

**Key Principle:** Pre-built libraries + smart loading = zero build time for users.

---

**Version:** 2.0  
**Last Updated:** 2025-11-28  
**Reference:** Ticket #100 (Python bindings)  
**Contact:** ask@vvoss.dev
