# Ticket #004: Rust Reference Implementation - Core Parser

**Repo**: gbln-rust  
**Status**: open  
**Priority**: critical  
**Created**: 2025-01-22  
**Updated**: 2025-01-22  

---

## Summary

Implement the reference parser for GBLN in Rust. This is the foundation for all other language bindings via C FFI.

---

## Requirements

### Core Functionality

1. **Lexer/Tokenizer**
   - Strip comments (`:| ...`)
   - Tokenize identifiers, type hints, delimiters
   - Track line/column numbers for error reporting

2. **Parser**
   - Recursive descent parser
   - Single-pass parsing
   - Build `Value` tree from tokens

3. **Type System**
   - All integer types: i8, i16, i32, i64, u8, u16, u32, u64
   - All float types: f32, f64
   - All string types: s2, s4, s8, s16, s32, s64, s128, s256, s512, s1024
   - Boolean: b (t/f/true/false/0/1)
   - Null: n (empty/null/n)

4. **Validation**
   - Parse-time type validation
   - Integer range checking
   - String length checking (UTF-8 character count)
   - Duplicate key detection

5. **Error Handling**
   - Detailed error messages with line/column
   - Helpful suggestions
   - Error type categorization

6. **Serialization**
   - Value → GBLN string (compact)
   - Value → GBLN string (formatted/pretty)

---

## File Structure

```
core/rust/gbln/
├── Cargo.toml
├── src/
│   ├── lib.rs              # Public API
│   ├── lexer.rs            # Tokenization
│   ├── parser.rs           # Parsing logic
│   ├── types.rs            # Type system
│   ├── value.rs            # Value representation
│   ├── validator.rs        # Validation rules
│   ├── error.rs            # Error handling
│   ├── serializer.rs       # GBLN output
│   └── formatter.rs        # Pretty printing
├── tests/
│   ├── parse_test.rs
│   ├── validate_test.rs
│   ├── serialize_test.rs
│   └── fixtures/
│       ├── valid/
│       └── invalid/
└── benches/
    └── parse_bench.rs
```

---

## Dependencies

Minimal dependencies strategy:

```toml
[dependencies]
# No serde initially - manual parsing
# No regex - manual pattern matching

[dev-dependencies]
criterion = "0.5"  # Benchmarking
```

---

## Public API Design

```rust
// Parse GBLN string
pub fn parse(input: &str) -> Result<Value, Error>;

// Serialize to compact GBLN
pub fn to_string(value: &Value) -> String;

// Serialize to formatted GBLN
pub fn to_string_pretty(value: &Value) -> String;

// Main value type
pub enum Value {
    I8(i8), I16(i16), I32(i32), I64(i64),
    U8(u8), U16(u16), U32(u32), U64(u64),
    F32(f32), F64(f64),
    Str(String),
    Bool(bool),
    Null,
    Object(IndexMap<String, Value>),  // Preserves insertion order
    Array(Vec<Value>),
}

// Error type with rich information
pub struct Error {
    pub kind: ErrorKind,
    pub line: usize,
    pub column: usize,
    pub message: String,
    pub suggestion: Option<String>,
}
```

---

## Performance Targets

- **Parse speed**: ~65ms for 1000 records
- **Memory usage**: 70% smaller than equivalent JSON
- **Zero-copy where possible**: Use `&str` slices instead of `String` internally

---

## Testing Requirements

### Unit Tests (60%)
- Every validation rule
- Every type parsing
- Edge cases (empty strings, max values, UTF-8)

### Integration Tests (30%)
- Full document parsing
- Nested structures
- Error message quality

### Benchmark Tests (10%)
- Compare with JSON parsing (serde_json)
- Memory usage profiling

---

## Test Fixtures

Create shared test files:

```
tests/fixtures/valid/
  - employee-100.gbln
  - nested-objects.gbln
  - all-types.gbln
  - utf8-strings.gbln
  - empty-values.gbln

tests/fixtures/invalid/
  - integer-out-of-range.gbln
  - string-too-long.gbln
  - duplicate-keys.gbln
  - invalid-syntax.gbln
  - missing-type-hint.gbln
```

---

## Acceptance Criteria

- [ ] All tests pass (100% coverage on core logic)
- [ ] Parses 100-employee benchmark correctly
- [ ] Validates all type constraints at parse-time
- [ ] Error messages include line/column numbers
- [ ] Serialization round-trips correctly (parse → serialize → parse)
- [ ] Benchmarks show acceptable performance vs JSON
- [ ] Documentation with examples
- [ ] No unsafe code (except if absolutely necessary and documented)

---

## Timeline

**Estimated**: 2 weeks  
**Dependencies**: None  
**Blocks**: #005 (C FFI), #009 (Python), #010 (JavaScript)

---

## Notes

- Follow KISS principles: files <400 lines
- Use British English in comments
- No generic names (helpers.rs, utils.rs)
- Separate test files (no inline `#[cfg(test)]`)
- Single responsibility per function

---

## References

- Specification: `docs/01-specification.md`
- Rust implementation guide: `.claude/Import/docs/04-rust-implementation.md`
- CLAUDE.md development rules: `.claude/CLAUDE.md`
