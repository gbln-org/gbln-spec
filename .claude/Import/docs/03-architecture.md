# GBLN Project Architecture

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    GBLN Ecosystem                            │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐               │
│  │   Tools   │  │  Editors  │  │  Website  │               │
│  │  fmt/lint │  │VS/Vim/Zed │  │Playground │               │
│  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘               │
│        │              │              │                       │
│        └──────────────┴──────────────┘                       │
│                       │                                      │
├───────────────────────┼──────────────────────────────────────┤
│                       ▼                                      │
│              Language Bindings                               │
│  ┌────────┬────────┬────────┬────────┬────────┬────────┐   │
│  │Python  │  JS    │ Swift  │Kotlin  │  Go    │ Java   │   │
│  │        │ /Node  │        │        │        │        │   │
│  └───┬────┴───┬────┴───┬────┴───┬────┴───┬────┴───┬────┘   │
│      │        │        │        │        │        │         │
│  ┌───┴────┬───┴────┬───┴────┬───┴────┬───┴────┬───┴────┐   │
│  │  C#    │  C++   │ Ruby   │  PHP   │ Perl   │  Tcl   │   │
│  └───┬────┴───┬────┴───┬────┴───┬────┴───┬────┴───┬────┘   │
│      │        │        │        │        │        │         │
│      └────────┴────────┴────────┴────────┴────────┘         │
│                       │                                      │
│                       ▼                                      │
├───────────────────────────────────────────────────────────────┤
│                  FFI Layer (C)                               │
│  ┌────────────────────────────────────────────────────┐     │
│  │  libgbln.so / libgbln.dylib / gbln.dll             │     │
│  │                                                     │     │
│  │  • gbln_parse()                                     │     │
│  │  • gbln_free()                                      │     │
│  │  • gbln_serialize()                                 │     │
│  │  • gbln_validate()                                  │     │
│  └─────────────────────┬───────────────────────────────┘     │
│                        │                                     │
│                        ▼                                     │
├───────────────────────────────────────────────────────────────┤
│                 Core Parser (Rust)                           │
│  ┌────────────────────────────────────────────────────┐     │
│  │  • Lexer/Tokenizer                                  │     │
│  │  • Parser (recursive descent)                       │     │
│  │  • Type System                                      │     │
│  │  • Validator                                        │     │
│  │  • Error Handler                                    │     │
│  │  • Serializer                                       │     │
│  └────────────────────────────────────────────────────┘     │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## Core Components

### 1. Rust Core (Reference Implementation)

**Location**: `core/rust/`

**Purpose**: 
- Reference implementation
- Fastest, most memory-safe implementation
- Compiles to WebAssembly for browser
- Compiles to C FFI for other languages

**Modules**:

```rust
gbln/
├── src/
│   ├── lib.rs              // Public API
│   ├── lexer.rs            // Tokenization
│   ├── parser.rs           // Parsing logic
│   ├── types.rs            // Type system
│   ├── validator.rs        // Validation rules
│   ├── error.rs            // Error handling
│   ├── value.rs            // Value representation
│   └── serializer.rs       // GBLN generation
├── tests/
│   ├── unit/               // Unit tests
│   ├── integration/        // Integration tests
│   └── fixtures/           // Test data
├── benches/
│   ├── parse.rs            // Parse benchmarks
│   └── memory.rs           // Memory benchmarks
└── Cargo.toml
```

**Key Types**:

```rust
// Value representation
pub enum Value {
    I8(i8),
    I16(i16),
    I32(i32),
    I64(i64),
    U8(u8),
    U16(u16),
    U32(u32),
    U64(u64),
    F32(f32),
    F64(f64),
    Str(String),
    Bool(bool),
    Null,
    Object(HashMap<String, Value>),
    Array(Vec<Value>),
}

// Type hints
pub enum TypeHint {
    I8, I16, I32, I64,
    U8, U16, U32, U64,
    F32, F64,
    Str(usize),  // max characters
    Bool,
    Null,
}

// Error types
pub enum ParseError {
    UnexpectedToken { expected: String, found: String, line: usize, col: usize },
    IntegerOutOfRange { value: String, type_hint: TypeHint, line: usize, col: usize },
    StringTooLong { actual: usize, max: usize, line: usize, col: usize },
    TypeMismatch { expected: TypeHint, found: String, line: usize, col: usize },
    DuplicateKey { key: String, first_line: usize, dup_line: usize },
    IoError(std::io::Error),
}
```

**Public API**:

```rust
// Parse GBLN string
pub fn parse(input: &str) -> Result<Value, ParseError>;

// Serialize to GBLN string
pub fn serialize(value: &Value) -> String;

// Pretty-print with indentation
pub fn serialize_pretty(value: &Value, indent: usize) -> String;

// Validate without full parsing
pub fn validate(input: &str) -> Result<(), ParseError>;

// Streaming parser for large files
pub struct StreamParser<R: Read> { /* ... */ }
impl<R: Read> StreamParser<R> {
    pub fn new(reader: R) -> Self;
    pub fn next_value(&mut self) -> Result<Option<Value>, ParseError>;
}
```

---

### 2. C FFI Layer

**Location**: `core/c/`

**Purpose**:
- Universal foreign function interface
- Bridge between Rust and other languages
- Stable C ABI for maximum compatibility

**Header** (`gbln.h`):

```c
#ifndef GBLN_H
#define GBLN_H

#include <stdint.h>
#include <stdbool.h>

#ifdef __cplusplus
extern "C" {
#endif

// Opaque pointer to GBLN value
typedef struct gbln_value gbln_value_t;

// Type enumeration
typedef enum {
    GBLN_TYPE_I8,
    GBLN_TYPE_I16,
    GBLN_TYPE_I32,
    GBLN_TYPE_I64,
    GBLN_TYPE_U8,
    GBLN_TYPE_U16,
    GBLN_TYPE_U32,
    GBLN_TYPE_U64,
    GBLN_TYPE_F32,
    GBLN_TYPE_F64,
    GBLN_TYPE_STR,
    GBLN_TYPE_BOOL,
    GBLN_TYPE_NULL,
    GBLN_TYPE_OBJECT,
    GBLN_TYPE_ARRAY,
} gbln_type_t;

// Error structure
typedef struct {
    int code;
    char message[256];
    size_t line;
    size_t column;
} gbln_error_t;

// Parse GBLN string
gbln_value_t* gbln_parse(const char* input, gbln_error_t* error);

// Free GBLN value
void gbln_free(gbln_value_t* value);

// Get value type
gbln_type_t gbln_type(const gbln_value_t* value);

// Extract values (return 0 on success, -1 on error)
int gbln_as_i8(const gbln_value_t* value, int8_t* out);
int gbln_as_u32(const gbln_value_t* value, uint32_t* out);
int gbln_as_str(const gbln_value_t* value, const char** out, size_t* len);
int gbln_as_bool(const gbln_value_t* value, bool* out);

// Object operations
gbln_value_t* gbln_object_get(const gbln_value_t* obj, const char* key);
size_t gbln_object_len(const gbln_value_t* obj);
int gbln_object_keys(const gbln_value_t* obj, const char*** keys, size_t* count);

// Array operations
gbln_value_t* gbln_array_get(const gbln_value_t* arr, size_t index);
size_t gbln_array_len(const gbln_value_t* arr);

// Serialization
char* gbln_serialize(const gbln_value_t* value);
char* gbln_serialize_pretty(const gbln_value_t* value, size_t indent);
void gbln_free_string(char* str);

// Validation
int gbln_validate(const char* input, gbln_error_t* error);

#ifdef __cplusplus
}
#endif

#endif // GBLN_H
```

**Implementation** (`gbln.c`):

```c
#include "gbln.h"
#include <stdlib.h>
#include <string.h>

// Forward declarations
extern void* rust_gbln_parse(const char*, void*);
extern void rust_gbln_free(void*);
// ... other extern declarations

gbln_value_t* gbln_parse(const char* input, gbln_error_t* error) {
    // Call Rust implementation through FFI
    return (gbln_value_t*)rust_gbln_parse(input, error);
}

void gbln_free(gbln_value_t* value) {
    if (value) {
        rust_gbln_free(value);
    }
}

// ... other implementations
```

---

### 3. Language Bindings

Each language binding provides idiomatic API while using the C FFI underneath.

#### Python (`bindings/python/`)

```python
# gbln/__init__.py
from typing import Union, Dict, List, Any

GBLNValue = Union[
    int, float, str, bool, None,
    Dict[str, 'GBLNValue'],
    List['GBLNValue']
]

def parse(text: str) -> GBLNValue:
    """Parse GBLN string into Python objects."""
    
def serialize(value: GBLNValue) -> str:
    """Serialize Python objects to GBLN string."""
    
def validate(text: str) -> bool:
    """Validate GBLN string without parsing."""

class ParseError(Exception):
    """GBLN parse error."""
    def __init__(self, message: str, line: int, column: int):
        self.message = message
        self.line = line
        self.column = column
```

#### JavaScript (`bindings/javascript/`)

```typescript
// index.d.ts
export type GBLNValue =
    | number
    | string
    | boolean
    | null
    | { [key: string]: GBLNValue }
    | GBLNValue[];

export function parse(input: string): GBLNValue;
export function serialize(value: GBLNValue): string;
export function validate(input: string): boolean;

export class ParseError extends Error {
    line: number;
    column: number;
    constructor(message: string, line: number, column: number);
}
```

#### Swift (`bindings/swift/`)

```swift
// GBLN.swift
public enum GBLNValue {
    case i8(Int8)
    case i16(Int16)
    case i32(Int32)
    case i64(Int64)
    case u8(UInt8)
    case u16(UInt16)
    case u32(UInt32)
    case u64(UInt64)
    case f32(Float)
    case f64(Double)
    case string(String)
    case bool(Bool)
    case null
    case object([String: GBLNValue])
    case array([GBLNValue])
}

public class GBLN {
    public static func parse(_ input: String) throws -> GBLNValue
    public static func serialize(_ value: GBLNValue) -> String
    public static func validate(_ input: String) -> Bool
}

public struct ParseError: Error {
    public let message: String
    public let line: Int
    public let column: Int
}
```

#### Kotlin (`bindings/kotlin/`)

```kotlin
// GBLN.kt
sealed class GBLNValue {
    data class I8(val value: Byte) : GBLNValue()
    data class I16(val value: Short) : GBLNValue()
    data class I32(val value: Int) : GBLNValue()
    data class I64(val value: Long) : GBLNValue()
    data class U8(val value: UByte) : GBLNValue()
    data class U16(val value: UShort) : GBLNValue()
    data class U32(val value: UInt) : GBLNValue()
    data class U64(val value: ULong) : GBLNValue()
    data class F32(val value: Float) : GBLNValue()
    data class F64(val value: Double) : GBLNValue()
    data class Str(val value: String) : GBLNValue()
    data class Bool(val value: Boolean) : GBLNValue()
    object Null : GBLNValue()
    data class Object(val value: Map<String, GBLNValue>) : GBLNValue()
    data class Array(val value: List<GBLNValue>) : GBLNValue()
}

object GBLN {
    @Throws(ParseException::class)
    fun parse(input: String): GBLNValue
    
    fun serialize(value: GBLNValue): String
    
    fun validate(input: String): Boolean
}

class ParseException(
    message: String,
    val line: Int,
    val column: Int
) : Exception(message)
```

---

## Parser Architecture

### Parsing Pipeline

```
Input String
    ↓
┌─────────────────┐
│ Comment Stripper│  Remove :| comments
└────────┬────────┘
         ↓
┌─────────────────┐
│     Lexer       │  Tokenize: {, }, <, >, (, ), [, ], identifiers, values
└────────┬────────┘
         ↓
┌─────────────────┐
│     Parser      │  Build syntax tree (recursive descent)
└────────┬────────┘
         ↓
┌─────────────────┐
│   Validator     │  Check types, ranges, lengths
└────────┬────────┘
         ↓
┌─────────────────┐
│ Value Builder   │  Create final Value tree
└────────┬────────┘
         ↓
    Result<Value>
```

### Comment Stripping (Preprocessing)

```rust
fn strip_comments(input: &str) -> String {
    input
        .lines()
        .map(|line| {
            if let Some(pos) = line.find(":|") {
                &line[..pos]
            } else {
                line
            }
        })
        .collect::<Vec<_>>()
        .join("\n")
}
```

### Lexer (Tokenization)

```rust
#[derive(Debug, Clone, PartialEq)]
pub enum Token {
    // Structural
    LeftBrace,      // {
    RightBrace,     // }
    LeftBracket,    // [
    RightBracket,   // ]
    LeftParen,      // (
    RightParen,     // )
    LeftAngle,      // <
    RightAngle,     // >
    
    // Values
    Identifier(String),
    TypeHint(TypeHint),
    Value(String),
    
    // Special
    Eof,
}

pub struct Lexer<'a> {
    input: &'a str,
    pos: usize,
    line: usize,
    column: usize,
}

impl<'a> Lexer<'a> {
    pub fn new(input: &'a str) -> Self;
    pub fn next_token(&mut self) -> Result<Token, ParseError>;
    pub fn peek(&self) -> Option<char>;
    fn skip_whitespace(&mut self);
    fn read_identifier(&mut self) -> String;
    fn read_type_hint(&mut self) -> Result<TypeHint, ParseError>;
    fn read_value(&mut self) -> Result<String, ParseError>;
}
```

### Parser (Recursive Descent)

```rust
pub struct Parser<'a> {
    lexer: Lexer<'a>,
    current: Token,
}

impl<'a> Parser<'a> {
    pub fn new(input: &'a str) -> Result<Self, ParseError>;
    
    pub fn parse(&mut self) -> Result<Value, ParseError>;
    
    fn parse_value(&mut self) -> Result<Value, ParseError>;
    fn parse_object(&mut self) -> Result<Value, ParseError>;
    fn parse_array(&mut self) -> Result<Value, ParseError>;
    fn parse_key_value(&mut self) -> Result<(String, Value), ParseError>;
    
    fn expect(&mut self, token: Token) -> Result<(), ParseError>;
    fn advance(&mut self) -> Result<(), ParseError>;
}
```

### Type-Driven Validation

```rust
fn parse_typed_value(raw: &str, type_hint: TypeHint) -> Result<Value, ParseError> {
    match type_hint {
        TypeHint::I8 => {
            let num = raw.trim().parse::<i64>()
                .map_err(|_| ParseError::InvalidInteger(raw.to_string()))?;
            
            if num < i8::MIN as i64 || num > i8::MAX as i64 {
                return Err(ParseError::IntegerOutOfRange {
                    value: raw.to_string(),
                    type_hint: TypeHint::I8,
                    line: 0, col: 0,
                });
            }
            
            Ok(Value::I8(num as i8))
        }
        
        TypeHint::Str(max_chars) => {
            let char_count = raw.chars().count();
            if char_count > max_chars {
                return Err(ParseError::StringTooLong {
                    actual: char_count,
                    max: max_chars,
                    line: 0, col: 0,
                });
            }
            Ok(Value::Str(raw.to_string()))
        }
        
        TypeHint::Bool => {
            let trimmed = raw.trim().to_lowercase();
            match trimmed.as_str() {
                "t" | "true" | "1" => Ok(Value::Bool(true)),
                "f" | "false" | "0" => Ok(Value::Bool(false)),
                _ => Err(ParseError::InvalidBoolean(raw.to_string())),
            }
        }
        
        // ... other types
    }
}
```

---

## Testing Strategy

### Test Pyramid

```
        ┌─────────────┐
        │   E2E Tests │  Cross-language compatibility
        │   (10%)     │  
        └──────┬──────┘
               │
        ┌──────┴──────┐
        │ Integration │   Full parsing + validation
        │   (30%)     │   
        └──────┬──────┘
               │
        ┌──────┴──────┐
        │ Unit Tests  │   Individual functions
        │   (60%)     │   Lexer, parser, validator
        └─────────────┘
```

### Test Categories

1. **Unit Tests**: Individual functions
   - Lexer tokenization
   - Parser grammar rules
   - Type validation
   - Error message formatting

2. **Integration Tests**: Complete workflows
   - Parse valid GBLN
   - Reject invalid GBLN
   - Round-trip (parse → serialize → parse)
   - Error recovery

3. **Cross-Language Tests**: Compatibility
   - Same input → same output across languages
   - Shared test fixture files
   - Performance comparison

4. **Benchmark Tests**: Performance
   - Parse speed
   - Memory usage
   - Comparison with JSON/YAML/TOML

---

## Build & Distribution

### Rust Core

```toml
# Cargo.toml
[package]
name = "gbln"
version = "0.1.0"

[lib]
crate-type = ["cdylib", "rlib"]

[dependencies]
# Minimal dependencies

[dev-dependencies]
criterion = "0.5"

[profile.release]
opt-level = 3
lto = true
```

### C FFI

```makefile
# Makefile
CC = gcc
CFLAGS = -Wall -Wextra -O3 -fPIC
LDFLAGS = -shared

libgbln.so: gbln.o rust_ffi.o
	$(CC) $(LDFLAGS) -o $@ $^

install:
	cp libgbln.so /usr/local/lib/
	cp gbln.h /usr/local/include/
```

### Package Managers

- **Rust**: cargo publish
- **Python**: pip / PyPI
- **JavaScript**: npm / yarn
- **Go**: go get
- **Swift**: CocoaPods / Swift Package Manager
- **Kotlin**: Maven Central / Gradle
- **Java**: Maven Central
- **C#**: NuGet
- **Ruby**: RubyGems
- **PHP**: Packagist

---

## Deployment Strategy

### Phase 1: Core Release
1. Rust crate published
2. C library compiled for Linux/macOS/Windows
3. Basic documentation

### Phase 2: Tier 1 Languages
1. Python, JS, Swift, Kotlin, Go
2. Package manager releases
3. API documentation

### Phase 3: Tier 2 Languages
1. Java, C#, C++
2. Enterprise support
3. Advanced examples

### Phase 4: Ecosystem
1. CLI tools
2. Editor plugins
3. Website + playground
4. Community building

---

*GBLN Project Architecture v1.0*
