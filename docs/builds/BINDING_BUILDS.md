# GBLN Language Bindings - Build & Test Guide

**Version:** 3.0  
**Last Updated:** 2025-12-01  
**Status:** Production - VERIFIED & TESTED

---

## Overview

GBLN provides bindings for 12+ languages, all using the pre-built C FFI libraries as foundation:

```
Rust Core → C FFI (cross builds) → Pre-built libs (Git) → Language Bindings → User API
```

**All bindings share:**
- Same C FFI libraries from `core/ffi/libs/` (committed to Git)
- Same platform support (8 platforms - ALL MANDATORY)
- Same quality requirements (tested on all platforms)
- Same documentation patterns

---

## ⚠️ NEW: Binding Categories (2025-12-01)

**CRITICAL: There are TWO distinct categories of language bindings:**

### Category 1: FS-Enabled Bindings (Full Feature Set)

**Languages:** Python, Swift, Kotlin, Go, Java, C#, C++, Ruby, PHP, Perl, Tcl

**Feature Set:**
- ✅ Parser (parse strings AND files)
- ✅ Serialiser (toString, toStringPretty)
- ✅ **IO Module (readFile, writeFile)** - MANDATORY
- ✅ Value conversion
- ✅ Config module
- ✅ Error types

**Use Case:** Native applications, servers, CLI tools, desktop apps

### Category 2: FS-Disabled Bindings (Limited Feature Set)

**Languages:** JavaScript (browser), TypeScript (browser), any WASM-only binding

**Feature Set:**
- ✅ Parser (parse strings ONLY)
- ✅ Serialiser (toString, toStringPretty)
- ✅ Value conversion
- ✅ Error types
- ❌ **NO IO Module** (browser security prevents filesystem access)
- ⚠️ Config module optional (only if needed)

**Use Case:** Browser applications, WASM environments without filesystem

**Quality Standards:** See `BINDING_BUILDS_QS_NOTES.md` for detailed requirements per category

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

**Every binding MUST support all 8 platforms - ALL MANDATORY.**

| Platform | Architecture | Library | Size | Support Level |
|----------|-------------|---------|------|---------------|
| Linux | x86_64 | libgbln.so | 573K | ✅ Mandatory |
| Linux | ARM64 | libgbln.so | 583K | ✅ Mandatory |
| FreeBSD | x86_64 | libgbln.so | 548K | ✅ Mandatory |
| FreeBSD | ARM64 | libgbln.so | 639K | ✅ Mandatory |
| Windows | x86_64 | gbln.dll | 1.5M | ✅ Mandatory |
| Android | ARM64 | libgbln.so | 627K | ✅ Mandatory |
| Android | x86_64 | libgbln.so | 617K | ✅ Mandatory |
| macOS | ARM64 | libgbln.dylib | 553K | ✅ Mandatory |

**Note:** FS-disabled bindings (JavaScript/WASM) still must handle all 8 platforms for Node.js compatibility

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

### Python (#100) - VERIFIED & TESTED

**Status:** ✅ Complete - 44/44 tests passing (2025-11-28)

**Key Implementation Details:**

1. **Library Loading:**
   ```python
   # CRITICAL: Use .resolve() on Path to handle pytest's relative paths
   package_dir = Path(__file__).parent.resolve()
   libs_base = package_dir.parent.parent.parent.parent / "core" / "ffi" / "libs"
   lib_path = libs_base / lib_dir / lib_name
   ```
   
   **Why `.resolve()`?** pytest loads modules from `tests/../src/gbln/` which creates relative paths. Without `.resolve()`, path traversal calculates incorrectly and libraries aren't found.

2. **Memory Management:**
   ```python
   import weakref
   
   class Value:
       def __init__(self, ptr):
           self._ptr = ptr
           # Automatically free when garbage collected
           weakref.finalize(self, _lib.gbln_value_free, ptr)
   ```

3. **Type Hints:**
   - All public functions fully typed
   - Return types explicit: `-> dict[str, Any]`, `-> str`, etc.
   - Parameters typed: `input: str`, `config: GblnConfig`, etc.

4. **Testing:**
   - 44 tests across 4 test files
   - Use `pytest` with `-o addopts=""` to override config
   - Clear Python cache before tests: `find . -name __pycache__ -exec rm -rf {} +`

**Common Issues & Solutions:**

**Issue 1: "No precompiled binary available" when running pytest**

**Cause:** Path calculation uses relative paths without `.resolve()`

**Solution:**
```python
# WRONG - fails with pytest
package_dir = Path(__file__).parent

# CORRECT - works with pytest
package_dir = Path(__file__).parent.resolve()
```

**Issue 2: "Illegal instruction: 4" or "Killed: 9" on macOS**

**Cause:** macOS library has wrong install_name (absolute path instead of @rpath)

**Solution:**
```bash
# Check install_name
otool -L core/ffi/libs/macos-arm64/libgbln.dylib

# Fix if needed
install_name_tool -id @rpath/libgbln.dylib core/ffi/libs/macos-arm64/libgbln.dylib
```

**Issue 3: Tests pass with `python3 -c` but fail with pytest**

**Cause:** pytest caches compiled `.pyc` files

**Solution:**
```bash
# Clear all Python caches
find . -type d -name __pycache__ -exec rm -rf {} + 2>/dev/null || true
find . -name "*.pyc" -delete
rm -rf .pytest_cache

# Then run tests
pytest -v
```

**Issue 4: Editable install not picking up changes**

**Cause:** pip caches the old installation

**Solution:**
```bash
# Uninstall and reinstall
python3 -m pip uninstall -y gbln
python3 -m pip install -e .

# Verify installation
python3 -m pip show gbln
```

**Lessons Learned (2025-11-28):**

1. ✅ **Always use `.resolve()`** on `Path(__file__).parent` to handle pytest's relative imports
2. ✅ **Verify macOS install_name** after every C FFI rebuild - wrong install_name causes immediate crashes
3. ✅ **Clear Python caches** before testing - stale `.pyc` files cause mysterious failures
4. ✅ **Test both ways** - Run tests with `python3 -c` AND `pytest` to catch path issues early
5. ✅ **Use editable install** for development - `pip install -e .` makes testing faster

**Performance:**
- Library loads in <1ms
- Parse performance: ~10,000 records/second
- 44 tests complete in <100ms

**Files:**
- `bindings/python/src/gbln/ffi.py` - FFI layer with platform detection
- `bindings/python/src/gbln/parse.py` - Parsing API
- `bindings/python/src/gbln/value.py` - Value wrapper with memory management
- `bindings/python/tests/` - 44 tests across 4 files

**Use Python bindings as reference for ALL other language bindings.**

### JavaScript (#101)
- Use `ffi-napi` for FFI or WASM
- Use `FinalizationRegistry` for memory
- TypeScript definitions required

### Swift (#102)
- Use `dlopen()` + Swift/C interop
- Use `deinit` for cleanup
- Native feel with Swift naming conventions

### Kotlin (#103) - VERIFIED & TESTED (FS-Enabled)

**Status:** ✅ Core Complete, ⚠️ Missing modules (2025-12-01)

**Key Implementation Details:**

1. **FFI Method:** Use JNA (Java Native Access), NOT JNI
   ```kotlin
   import com.sun.jna.Library
   import com.sun.jna.Native
   
   interface GblnLibrary : Library {
       fun gbln_parse(input: String, outValue: PointerByReference): Int
       // ... all C FFI functions
   }
   ```

2. **Memory Management:**
   ```kotlin
   class ManagedGblnValue(val ptr: Pointer) {
       init {
           cleaner.register(this, CleanupAction(ptr))
       }
       private class CleanupAction(private val ptr: Pointer) : Runnable {
           override fun run() {
               lib.gblnValueFree(ptr)
           }
       }
   }
   ```

3. **Build System:** Gradle with JNA dependency
   ```kotlin
   dependencies {
       implementation("net.java.dev.jna:jna:5.14.0")
   }
   ```

**Missing Modules (see BINDING_BUILDS_QS_NOTES.md):**
- ❌ io.kt (CRITICAL - mandatory for FS-enabled)
- ❌ Config.kt
- ❌ Error.kt
- ⚠️ Incomplete test coverage

**Reference:** Python bindings for structure, adapt to Kotlin idioms

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

1. ✅ **Determine category** - FS-enabled (full) or FS-disabled (browser)
2. ✅ Use pre-built libraries from `core/ffi/libs/` (DO NOT rebuild)
3. ✅ Implement platform detection + library loading
4. ✅ Include all 8 libraries in package distribution
5. ✅ Implement ALL mandatory modules per category
6. ✅ Test on your platform (auto-detects correct library)
7. ✅ Users install package and it just works (no compilation)

**Key Principle:** Pre-built libraries + smart loading = zero build time for users.

**Quality Assurance:** Follow `BINDING_BUILDS_QS_NOTES.md` for complete requirements.

---

## Related Documentation

- **`BUILD_SYSTEM.md`** - How C FFI libraries are built
- **`BINDING_BUILDS_QS_NOTES.md`** - Quality standards and checklists per category
- **`.claude/tickets/100-gbln-bindings-python-STATUS.md`** - Reference implementation (FS-enabled)
- **`.claude/tickets/101-gbln-bindings-javascript.md`** - Reference implementation (FS-disabled)
- **`.claude/tickets/103-gbln-bindings-kotlin.md`** - Current work (FS-enabled, incomplete)

---

**Version:** 3.0  
**Last Updated:** 2025-12-01  
**Reference:** Ticket #100 (Python - FS-enabled), Ticket #101 (JavaScript - FS-disabled)  
**Contact:** ask@vvoss.dev
