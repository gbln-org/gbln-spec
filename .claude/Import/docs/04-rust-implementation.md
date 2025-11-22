# Rust Implementation Guide

## Overview

The Rust implementation serves as the **reference implementation** for GBLN. It provides:
- Fastest parsing performance
- Memory-safe operations (zero-cost abstractions)
- WebAssembly compilation for browsers
- C FFI for other language bindings
- Comprehensive test suite

## Project Structure

```
core/rust/gbln/
├── Cargo.toml
├── src/
│   ├── lib.rs              # Public API & re-exports
│   ├── lexer.rs            # Tokenization
│   ├── parser.rs           # Recursive descent parser
│   ├── types.rs            # Type system & type hints
│   ├── value.rs            # Value representation
│   ├── validator.rs        # Validation logic
│   ├── error.rs            # Error types & messages
│   ├── serializer.rs       # GBLN generation
│   └── ffi.rs              # C FFI bindings
├── tests/
│   ├── unit/
│   │   ├── lexer_test.rs
│   │   ├── parser_test.rs
│   │   └── validator_test.rs
│   ├── integration/
│   │   ├── parse_test.rs
│   │   └── roundtrip_test.rs
│   └── fixtures/
│       ├── valid/          # Valid GBLN files
│       └── invalid/        # Invalid GBLN files
├── benches/
│   ├── parse_bench.rs
│   ├── memory_bench.rs
│   └── comparison_bench.rs
└── examples/
    ├── basic.rs
    ├── streaming.rs
    └── error_handling.rs
```

---

## Cargo.toml

```toml
[package]
name = "gbln"
version = "0.1.0"
authors = ["Vivian Voss <ask@vvoss.dev>"]
edition = "2021"
license = "MIT"
description = "GBLN (Goblin Bounded Lean Notation) - Type-safe, memory-efficient serialization format"
repository = "https://github.com/gbln/gbln-rust"
keywords = ["serialization", "parser", "type-safe", "data-format"]
categories = ["parsing", "encoding"]

[lib]
name = "gbln"
crate-type = ["rlib", "cdylib"]  # Library + C FFI

[dependencies]
# Keep dependencies minimal for security & compile time
# No regex, no serde (initially) - pure Rust parser

[dev-dependencies]
criterion = "0.5"      # Benchmarking
proptest = "1.0"       # Property-based testing

[profile.release]
opt-level = 3
lto = true             # Link-time optimization
codegen-units = 1      # Better optimization
strip = true           # Strip symbols

[profile.bench]
inherits = "release"

[[bench]]
name = "parse"
harness = false

[[bench]]
name = "memory"
harness = false
```

---

## Core Types (types.rs)

```rust
// types.rs

/// Type hint for GBLN values
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub enum TypeHint {
    // Signed integers
    I8,
    I16,
    I32,
    I64,
    
    // Unsigned integers
    U8,
    U16,
    U32,
    U64,
    
    // Floats
    F32,
    F64,
    
    // String with max character count
    Str(usize),
    
    // Boolean
    Bool,
    
    // Null
    Null,
}

impl TypeHint {
    /// Parse type hint from string (e.g., "i8", "u32", "s64")
    pub fn from_str(s: &str) -> Result<Self, ParseError> {
        match s {
            "i8" => Ok(TypeHint::I8),
            "i16" => Ok(TypeHint::I16),
            "i32" => Ok(TypeHint::I32),
            "i64" => Ok(TypeHint::I64),
            "u8" => Ok(TypeHint::U8),
            "u16" => Ok(TypeHint::U16),
            "u32" => Ok(TypeHint::U32),
            "u64" => Ok(TypeHint::U64),
            "f32" => Ok(TypeHint::F32),
            "f64" => Ok(TypeHint::F64),
            "b" => Ok(TypeHint::Bool),
            "n" => Ok(TypeHint::Null),
            s if s.starts_with('s') => {
                let size = s[1..].parse::<usize>()
                    .map_err(|_| ParseError::InvalidTypeHint(s.to_string()))?;
                Ok(TypeHint::Str(size))
            }
            _ => Err(ParseError::InvalidTypeHint(s.to_string())),
        }
    }
    
    /// Get display name for error messages
    pub fn display_name(&self) -> String {
        match self {
            TypeHint::I8 => "i8".to_string(),
            TypeHint::I16 => "i16".to_string(),
            TypeHint::I32 => "i32".to_string(),
            TypeHint::I64 => "i64".to_string(),
            TypeHint::U8 => "u8".to_string(),
            TypeHint::U16 => "u16".to_string(),
            TypeHint::U32 => "u32".to_string(),
            TypeHint::U64 => "u64".to_string(),
            TypeHint::F32 => "f32".to_string(),
            TypeHint::F64 => "f64".to_string(),
            TypeHint::Str(n) => format!("s{}", n),
            TypeHint::Bool => "b".to_string(),
            TypeHint::Null => "n".to_string(),
        }
    }
    
    /// Get valid range for integers (for error messages)
    pub fn int_range(&self) -> Option<(i64, i64)> {
        match self {
            TypeHint::I8 => Some((i8::MIN as i64, i8::MAX as i64)),
            TypeHint::I16 => Some((i16::MIN as i64, i16::MAX as i64)),
            TypeHint::I32 => Some((i32::MIN as i64, i32::MAX as i64)),
            TypeHint::I64 => Some((i64::MIN, i64::MAX)),
            TypeHint::U8 => Some((0, u8::MAX as i64)),
            TypeHint::U16 => Some((0, u16::MAX as i64)),
            TypeHint::U32 => Some((0, u32::MAX as i64)),
            TypeHint::U64 => Some((0, i64::MAX)), // Can't represent full u64 range in i64
            _ => None,
        }
    }
}
```

---

## Value Representation (value.rs)

```rust
// value.rs

use std::collections::HashMap;

/// GBLN value - tagged union for type-safe storage
#[derive(Debug, Clone, PartialEq)]
pub enum Value {
    // Integers
    I8(i8),
    I16(i16),
    I32(i32),
    I64(i64),
    U8(u8),
    U16(u16),
    U32(u32),
    U64(u64),
    
    // Floats
    F32(f32),
    F64(f64),
    
    // String
    Str(String),
    
    // Boolean
    Bool(bool),
    
    // Null
    Null,
    
    // Containers
    Object(HashMap<String, Value>),
    Array(Vec<Value>),
}

impl Value {
    /// Get the type of this value
    pub fn type_name(&self) -> &'static str {
        match self {
            Value::I8(_) => "i8",
            Value::I16(_) => "i16",
            Value::I32(_) => "i32",
            Value::I64(_) => "i64",
            Value::U8(_) => "u8",
            Value::U16(_) => "u16",
            Value::U32(_) => "u32",
            Value::U64(_) => "u64",
            Value::F32(_) => "f32",
            Value::F64(_) => "f64",
            Value::Str(_) => "string",
            Value::Bool(_) => "bool",
            Value::Null => "null",
            Value::Object(_) => "object",
            Value::Array(_) => "array",
        }
    }
    
    /// Check if value is a container (object or array)
    pub fn is_container(&self) -> bool {
        matches!(self, Value::Object(_) | Value::Array(_))
    }
    
    /// Try to get value as i8
    pub fn as_i8(&self) -> Option<i8> {
        match self {
            Value::I8(v) => Some(*v),
            _ => None,
        }
    }
    
    /// Try to get value as string slice
    pub fn as_str(&self) -> Option<&str> {
        match self {
            Value::Str(s) => Some(s.as_str()),
            _ => None,
        }
    }
    
    /// Try to get value as object
    pub fn as_object(&self) -> Option<&HashMap<String, Value>> {
        match self {
            Value::Object(obj) => Some(obj),
            _ => None,
        }
    }
    
    /// Try to get value as array
    pub fn as_array(&self) -> Option<&Vec<Value>> {
        match self {
            Value::Array(arr) => Some(arr),
            _ => None,
        }
    }
    
    // Add similar methods for other types...
}

// Implement Index for convenient access
use std::ops::Index;

impl Index<&str> for Value {
    type Output = Value;
    
    fn index(&self, key: &str) -> &Self::Output {
        match self {
            Value::Object(obj) => obj.get(key)
                .unwrap_or(&Value::Null),
            _ => &Value::Null,
        }
    }
}

impl Index<usize> for Value {
    type Output = Value;
    
    fn index(&self, index: usize) -> &Self::Output {
        match self {
            Value::Array(arr) => arr.get(index)
                .unwrap_or(&Value::Null),
            _ => &Value::Null,
        }
    }
}
```

---

## Error Types (error.rs)

```rust
// error.rs

use std::fmt;

/// Parse error with position information
#[derive(Debug, Clone, PartialEq)]
pub enum ParseError {
    /// Unexpected token
    UnexpectedToken {
        expected: String,
        found: String,
        line: usize,
        column: usize,
    },
    
    /// Integer out of range for type
    IntegerOutOfRange {
        value: String,
        type_hint: crate::TypeHint,
        line: usize,
        column: usize,
    },
    
    /// String exceeds maximum length
    StringTooLong {
        actual: usize,
        max: usize,
        line: usize,
        column: usize,
    },
    
    /// Type mismatch
    TypeMismatch {
        expected: crate::TypeHint,
        found: String,
        line: usize,
        column: usize,
    },
    
    /// Duplicate key in object
    DuplicateKey {
        key: String,
        first_line: usize,
        dup_line: usize,
    },
    
    /// Invalid type hint
    InvalidTypeHint(String),
    
    /// Invalid integer literal
    InvalidInteger(String),
    
    /// Invalid float literal
    InvalidFloat(String),
    
    /// Invalid boolean literal
    InvalidBoolean(String),
    
    /// Unexpected end of input
    UnexpectedEof,
    
    /// IO error
    IoError(String),
}

impl fmt::Display for ParseError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            ParseError::UnexpectedToken { expected, found, line, column } => {
                write!(f, "Error: Unexpected token\n")?;
                write!(f, "  expected: {}\n", expected)?;
                write!(f, "  found: {}\n", found)?;
                write!(f, "  line: {}\n", line)?;
                write!(f, "  column: {}\n", column)?;
            }
            
            ParseError::IntegerOutOfRange { value, type_hint, line, column } => {
                write!(f, "Error: Integer out of range\n")?;
                write!(f, "  value: {}\n", value)?;
                write!(f, "  type: {}\n", type_hint.display_name())?;
                if let Some((min, max)) = type_hint.int_range() {
                    write!(f, "  valid range: {} to {}\n", min, max)?;
                }
                write!(f, "  line: {}\n", line)?;
                write!(f, "  column: {}\n", column)?;
                write!(f, "\n  suggestion: Use {} or {} for larger values",
                    suggest_larger_int(type_hint, true),
                    suggest_larger_int(type_hint, false))?;
            }
            
            ParseError::StringTooLong { actual, max, line, column } => {
                write!(f, "Error: String exceeds maximum length\n")?;
                write!(f, "  actual: {} characters\n", actual)?;
                write!(f, "  maximum: {} characters\n", max)?;
                write!(f, "  line: {}\n", line)?;
                write!(f, "  column: {}\n", column)?;
                write!(f, "\n  suggestion: Use s{} or s{} for longer strings",
                    max * 2, max * 4)?;
            }
            
            ParseError::DuplicateKey { key, first_line, dup_line } => {
                write!(f, "Error: Duplicate key in object\n")?;
                write!(f, "  key: \"{}\"\n", key)?;
                write!(f, "  first occurrence: line {}\n", first_line)?;
                write!(f, "  duplicate: line {}\n", dup_line)?;
                write!(f, "\n  suggestion: Remove duplicate key or rename one of them")?;
            }
            
            _ => write!(f, "{:?}", self)?,
        }
        Ok(())
    }
}

impl std::error::Error for ParseError {}

fn suggest_larger_int(hint: &crate::TypeHint, signed: bool) -> &'static str {
    use crate::TypeHint::*;
    match (hint, signed) {
        (I8, true) => "i16",
        (I8, false) => "i32",
        (I16, true) => "i32",
        (I16, false) => "i64",
        (I32, true) => "i64",
        (I32, false) => "i64",
        (U8, _) => "u16",
        (U16, _) => "u32",
        (U32, _) => "u64",
        _ => "larger type",
    }
}

/// Result type for GBLN operations
pub type Result<T> = std::result::Result<T, ParseError>;
```

---

## Lexer (lexer.rs)

```rust
// lexer.rs

use crate::{ParseError, Result, TypeHint};

/// Token types
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
    
    // Special
    Eof,
}

/// Lexer for tokenizing GBLN input
pub struct Lexer<'a> {
    input: &'a str,
    chars: std::iter::Peekable<std::str::CharIndices<'a>>,
    current_pos: usize,
    line: usize,
    column: usize,
}

impl<'a> Lexer<'a> {
    /// Create new lexer
    pub fn new(input: &'a str) -> Self {
        Self {
            input,
            chars: input.char_indices().peekable(),
            current_pos: 0,
            line: 1,
            column: 1,
        }
    }
    
    /// Get current position
    pub fn position(&self) -> (usize, usize) {
        (self.line, self.column)
    }
    
    /// Peek at next character without consuming
    fn peek(&mut self) -> Option<char> {
        self.chars.peek().map(|(_, ch)| *ch)
    }
    
    /// Advance to next character
    fn advance(&mut self) -> Option<char> {
        if let Some((pos, ch)) = self.chars.next() {
            self.current_pos = pos;
            if ch == '\n' {
                self.line += 1;
                self.column = 1;
            } else {
                self.column += 1;
            }
            Some(ch)
        } else {
            None
        }
    }
    
    /// Skip whitespace
    fn skip_whitespace(&mut self) {
        while let Some(ch) = self.peek() {
            if ch.is_whitespace() {
                self.advance();
            } else {
                break;
            }
        }
    }
    
    /// Read identifier
    fn read_identifier(&mut self) -> String {
        let start = self.current_pos;
        
        // First char must be letter or underscore
        if let Some(ch) = self.peek() {
            if ch.is_alphabetic() || ch == '_' {
                self.advance();
            }
        }
        
        // Rest can be letter, digit, or underscore
        while let Some(ch) = self.peek() {
            if ch.is_alphanumeric() || ch == '_' {
                self.advance();
            } else {
                break;
            }
        }
        
        self.input[start..=self.current_pos].to_string()
    }
    
    /// Read type hint
    fn read_type_hint(&mut self) -> Result<TypeHint> {
        let start = self.current_pos;
        
        // Read type base (i, u, f, s, b, n)
        let base = self.advance()
            .ok_or(ParseError::UnexpectedEof)?;
        
        // For string types, read size
        if base == 's' {
            let mut size_str = String::new();
            while let Some(ch) = self.peek() {
                if ch.is_numeric() {
                    size_str.push(ch);
                    self.advance();
                } else {
                    break;
                }
            }
            
            let size = size_str.parse::<usize>()
                .map_err(|_| ParseError::InvalidTypeHint(
                    self.input[start..=self.current_pos].to_string()
                ))?;
            
            return Ok(TypeHint::Str(size));
        }
        
        // For integer types, read bit width
        if base == 'i' || base == 'u' || base == 'f' {
            let mut width_str = String::new();
            while let Some(ch) = self.peek() {
                if ch.is_numeric() {
                    width_str.push(ch);
                    self.advance();
                } else {
                    break;
                }
            }
            
            let type_str = format!("{}{}", base, width_str);
            return TypeHint::from_str(&type_str);
        }
        
        // Boolean or null
        let type_str = base.to_string();
        TypeHint::from_str(&type_str)
    }
    
    /// Get next token
    pub fn next_token(&mut self) -> Result<Token> {
        self.skip_whitespace();
        
        let (line, column) = self.position();
        
        match self.peek() {
            None => Ok(Token::Eof),
            Some('{') => {
                self.advance();
                Ok(Token::LeftBrace)
            }
            Some('}') => {
                self.advance();
                Ok(Token::RightBrace)
            }
            Some('[') => {
                self.advance();
                Ok(Token::LeftBracket)
            }
            Some(']') => {
                self.advance();
                Ok(Token::RightBracket)
            }
            Some('(') => {
                self.advance();
                Ok(Token::LeftParen)
            }
            Some(')') => {
                self.advance();
                Ok(Token::RightParen)
            }
            Some('<') => {
                self.advance();
                // Check if this is a type hint
                let type_hint = self.read_type_hint()?;
                Ok(Token::TypeHint(type_hint))
            }
            Some('>') => {
                self.advance();
                Ok(Token::RightAngle)
            }
            Some(ch) if ch.is_alphabetic() || ch == '_' => {
                let ident = self.read_identifier();
                Ok(Token::Identifier(ident))
            }
            Some(ch) => {
                Err(ParseError::UnexpectedToken {
                    expected: "valid token".to_string(),
                    found: ch.to_string(),
                    line,
                    column,
                })
            }
        }
    }
}
```

---

## Public API (lib.rs)

```rust
// lib.rs

//! GBLN (Goblin Bounded Lean Notation) parser
//!
//! A type-safe, memory-efficient serialization format with parse-time validation.

mod lexer;
mod parser;
mod types;
mod value;
mod validator;
mod error;
mod serializer;

#[cfg(feature = "ffi")]
mod ffi;

// Public exports
pub use error::{ParseError, Result};
pub use types::TypeHint;
pub use value::Value;

/// Parse GBLN string into Value tree
///
/// # Examples
///
/// ```
/// let gbln = r#"
///     user{
///         id<u32>(12345)
///         name<s64>(Alice)
///         age<i8>(25)
///     }
/// "#;
///
/// let value = gbln::parse(gbln)?;
/// assert_eq!(value["user"]["id"].as_u32(), Some(12345));
/// ```
pub fn parse(input: &str) -> Result<Value> {
    // Strip comments first
    let cleaned = strip_comments(input);
    
    // Parse
    let mut parser = parser::Parser::new(&cleaned)?;
    parser.parse()
}

/// Serialize Value back to GBLN string
pub fn serialize(value: &Value) -> String {
    serializer::serialize(value)
}

/// Serialize with pretty printing
pub fn serialize_pretty(value: &Value, indent: usize) -> String {
    serializer::serialize_pretty(value, indent)
}

/// Validate GBLN without full parsing (faster)
pub fn validate(input: &str) -> Result<()> {
    let cleaned = strip_comments(input);
    validator::validate(&cleaned)
}

/// Strip comments from GBLN input
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

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_parse_simple() {
        let gbln = "user{id<u32>(123) name<s32>(Alice)}";
        let value = parse(gbln).unwrap();
        
        assert!(matches!(value, Value::Object(_)));
    }
    
    #[test]
    fn test_comments() {
        let gbln = r#"
            :| This is a comment
            user{
                id<u32>(123)  :| User ID
            }
        "#;
        
        let value = parse(gbln).unwrap();
        assert!(matches!(value, Value::Object(_)));
    }
}
```

---

## Next Steps

1. **Implement Parser** (`parser.rs`) - See [06-parser-architecture.md](06-parser-architecture.md)
2. **Implement Validator** (`validator.rs`)
3. **Implement Serializer** (`serializer.rs`)
4. **Add Tests** (unit + integration)
5. **Add Benchmarks**
6. **FFI Layer** (`ffi.rs`)

---

*Rust Implementation Guide v1.0*
