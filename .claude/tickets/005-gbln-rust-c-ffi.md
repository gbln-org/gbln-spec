# Ticket #005: C FFI Layer for Language Bindings

**Repo**: gbln-rust  
**Status**: open  
**Priority**: critical  
**Created**: 2025-01-22  
**Updated**: 2025-01-22  

---

## Summary

Create a C-compatible FFI layer on top of the Rust core to enable bindings for Python, JavaScript, Swift, Kotlin, Go, and other languages.

---

## Requirements

### Core Functionality

1. **C-Compatible API**
   - Export functions with C ABI (`#[no_mangle]`, `extern "C"`)
   - No Rust-specific types in signatures
   - C-compatible error handling

2. **Memory Management**
   - Clear ownership semantics
   - Allocation/deallocation functions
   - No memory leaks in C code

3. **String Handling**
   - UTF-8 C strings (null-terminated)
   - Buffer management for output

4. **Error Handling**
   - Error codes instead of Result<T, E>
   - Error message retrieval

---

## File Structure

```
core/rust/gbln-ffi/
├── Cargo.toml
├── src/
│   ├── lib.rs              # FFI exports
│   ├── types.rs            # C-compatible types
│   └── error.rs            # C error handling
├── include/
│   └── gbln.h              # C header file
├── tests/
│   └── ffi_test.c          # C integration tests
└── examples/
    └── example.c
```

---

## C API Design

### Header File (gbln.h)

```c
#ifndef GBLN_H
#define GBLN_H

#include <stdint.h>
#include <stdbool.h>

// Opaque handle to GBLN value
typedef struct GblnValue GblnValue;

// Error codes
typedef enum {
    GBLN_OK = 0,
    GBLN_ERROR_PARSE = 1,
    GBLN_ERROR_INVALID_TYPE = 2,
    GBLN_ERROR_OUT_OF_RANGE = 3,
    GBLN_ERROR_DUPLICATE_KEY = 4,
    GBLN_ERROR_NULL_POINTER = 5,
} GblnErrorCode;

// Parse GBLN string
GblnErrorCode gbln_parse(const char* input, GblnValue** out_value);

// Free value
void gbln_value_free(GblnValue* value);

// Serialize to string (compact)
char* gbln_to_string(const GblnValue* value);

// Serialize to string (formatted)
char* gbln_to_string_pretty(const GblnValue* value);

// Free string
void gbln_string_free(char* str);

// Get last error message
const char* gbln_last_error_message(void);

// Value accessors
int32_t gbln_value_as_i32(const GblnValue* value, bool* ok);
uint32_t gbln_value_as_u32(const GblnValue* value, bool* ok);
double gbln_value_as_f64(const GblnValue* value, bool* ok);
const char* gbln_value_as_string(const GblnValue* value, bool* ok);
bool gbln_value_as_bool(const GblnValue* value, bool* ok);

// Object accessors
const GblnValue* gbln_object_get(const GblnValue* value, const char* key);

// Array accessors
size_t gbln_array_len(const GblnValue* value);
const GblnValue* gbln_array_get(const GblnValue* value, size_t index);

#endif // GBLN_H
```

---

## Rust Implementation

### lib.rs (excerpt)

```rust
use std::ffi::{CStr, CString};
use std::os::raw::{c_char, c_int};
use std::ptr;
use gbln::{parse, Value};

// Opaque pointer to Value
pub struct GblnValue(Value);

// Error codes
#[repr(C)]
pub enum GblnErrorCode {
    Ok = 0,
    ErrorParse = 1,
    ErrorInvalidType = 2,
    ErrorOutOfRange = 3,
    ErrorDuplicateKey = 4,
    ErrorNullPointer = 5,
}

// Thread-local error storage
thread_local! {
    static LAST_ERROR: RefCell<Option<String>> = RefCell::new(None);
}

fn set_last_error(msg: String) {
    LAST_ERROR.with(|e| {
        *e.borrow_mut() = Some(msg);
    });
}

#[no_mangle]
pub extern "C" fn gbln_parse(
    input: *const c_char,
    out_value: *mut *mut GblnValue
) -> GblnErrorCode {
    if input.is_null() || out_value.is_null() {
        set_last_error("Null pointer".to_string());
        return GblnErrorCode::ErrorNullPointer;
    }
    
    let input_str = unsafe {
        match CStr::from_ptr(input).to_str() {
            Ok(s) => s,
            Err(e) => {
                set_last_error(format!("Invalid UTF-8: {}", e));
                return GblnErrorCode::ErrorParse;
            }
        }
    };
    
    match parse(input_str) {
        Ok(value) => {
            let boxed = Box::new(GblnValue(value));
            unsafe { *out_value = Box::into_raw(boxed); }
            GblnErrorCode::Ok
        }
        Err(e) => {
            set_last_error(e.to_string());
            GblnErrorCode::ErrorParse
        }
    }
}

#[no_mangle]
pub extern "C" fn gbln_value_free(value: *mut GblnValue) {
    if !value.is_null() {
        unsafe { Box::from_raw(value); }
    }
}

#[no_mangle]
pub extern "C" fn gbln_last_error_message() -> *const c_char {
    LAST_ERROR.with(|e| {
        let borrowed = e.borrow();
        match &*borrowed {
            Some(msg) => {
                let c_str = CString::new(msg.as_str()).unwrap();
                c_str.into_raw()
            }
            None => ptr::null()
        }
    })
}
```

---

## Build Configuration

### Cargo.toml

```toml
[package]
name = "gbln-ffi"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib", "staticlib"]

[dependencies]
gbln = { path = "../gbln" }

[build-dependencies]
cbindgen = "0.26"
```

### build.rs

```rust
extern crate cbindgen;

use std::env;

fn main() {
    let crate_dir = env::var("CARGO_MANIFEST_DIR").unwrap();
    
    cbindgen::Builder::new()
        .with_crate(crate_dir)
        .generate()
        .expect("Unable to generate bindings")
        .write_to_file("include/gbln.h");
}
```

---

## Testing Strategy

### C Integration Test (ffi_test.c)

```c
#include "gbln.h"
#include <stdio.h>
#include <assert.h>

void test_parse_simple() {
    const char* input = "user{id<u32>(12345)name<s32>(Alice)}";
    GblnValue* value = NULL;
    
    GblnErrorCode err = gbln_parse(input, &value);
    assert(err == GBLN_OK);
    
    const GblnValue* user = gbln_object_get(value, "user");
    const GblnValue* id = gbln_object_get(user, "id");
    
    bool ok;
    uint32_t id_val = gbln_value_as_u32(id, &ok);
    assert(ok == true);
    assert(id_val == 12345);
    
    gbln_value_free(value);
    printf("test_parse_simple: PASSED\n");
}

int main() {
    test_parse_simple();
    // More tests...
    return 0;
}
```

---

## Acceptance Criteria

- [ ] C header file generated automatically (cbindgen)
- [ ] Shared library builds (libgbln.so / libgbln.dylib / gbln.dll)
- [ ] No memory leaks (verified with valgrind)
- [ ] C integration tests pass
- [ ] Error handling works correctly
- [ ] Thread-safe (or documented as not thread-safe)
- [ ] Example C program works

---

## Timeline

**Estimated**: 1 week  
**Dependencies**: #004 (Rust core)  
**Blocks**: #009 (Python), #010 (JavaScript), all language bindings

---

## Platform Support

- **Linux**: libgbln.so
- **macOS**: libgbln.dylib
- **Windows**: gbln.dll

Build for all three platforms using CI/CD.

---

## Notes

- Use cbindgen for automatic header generation
- Keep C API minimal - language bindings can add convenience
- Document ownership rules clearly
- Consider thread-safety (TLS for error messages)

---

## References

- Rust FFI documentation: https://doc.rust-lang.org/nomicon/ffi.html
- cbindgen: https://github.com/eqrion/cbindgen
