# GBLN Project - Claude AI Assistant Guide

**Version**: 1.1  
**Last Updated**: 2025-01-21  
**Project Status**: Concept Phase - Ready for Implementation

---

## 🚨 MANDATORY DEVELOPMENT RULES 🚨

**These rules are UNBREAKABLE and apply to ALL code in this project.**

### Absolutely FORBIDDEN ❌

- ❌ **Simplifying error types** - "we only need 10 variants, not 40"
- ❌ **Omitting functions** - "this looks unused, let's skip it"
- ❌ **Reducing enum variants** - "we can combine these cases"
- ❌ **Skipping trait implementations** - "not needed right now"
- ❌ **Abbreviating implementations** - "we'll add this later"
- ❌ **"Modernising" or "improving"** anything without explicit approval

### What IS Allowed ✅

- ✅ **Suggesting improvements** - "This could be solved better with X, shall I?"
- ✅ **Proposing removals** - "Function Y is provably unused (grep shows 0 calls), remove?"
- ✅ **Better solutions** - "Pattern Z is cleaner than current approach, switch?"
- ✅ **Refactoring proposals** - "Duplicated code could be unified, proceed?"

---

### Standard #0: NEVER INVENT - Always Ask First

**GOLDEN RULE: If you don't know something, say "I don't know - show me" instead of inventing it!**

- ❌ **NEVER** invent syntax, specifications, or features for external formats/tools
- ❌ **NEVER** guess how third-party libraries or formats work
- ❌ **NEVER** make up comparisons or statistics without verification
- ✅ **ALWAYS** say "I don't know X - can you show me?" when unfamiliar
- ✅ **ALWAYS** use WebSearch to verify external information before using it
- ✅ **ALWAYS** admit knowledge gaps instead of fabricating

**Example:**
- User: "Compare with TOON format"
- ❌ Wrong: *invents TOON syntax and makes up statistics*
- ✅ Right: "I'm not familiar with TOON's exact syntax - could you show me documentation or should I search for it?"

### Standard #1: Code Reuse

**NEVER duplicate existing functions**

- Check: `grep "function_name" _workbench/analysis/050-all-functions.txt` (when available)
- Use: Existing functions from `current/` or `last/` directories
- If similar exists: Extend it, don't duplicate

### Standard #2: BBC English

**ALL comments and documentation in British English**

- ✅ `initialise`, `optimise`, `analyse`, `behaviour`, `colour`, `serialise`, `tokenise`
- ❌ `initialize`, `optimize`, `analyze`, `behavior`, `color`, `serialize`, `tokenize`
- **Exception**: Code identifiers from ecosystem (e.g., `serialize` trait from serde)

### Standard #3: KISS - File Size <400 Lines

**Keep files simple and focused**

- Check: `wc -l src/module/file.rs`
- If >400 lines: Split into multiple files
- Each file = ONE clear responsibility
- Large modules use subdirectories with `mod.rs`

### Standard #4: File Naming

**Specific names, not generic**

- ✅ `path_construction.rs`, `key_validation.rs`, `integer_range_validator.rs`
- ❌ `helpers.rs`, `utils.rs`, `common.rs`, `misc.rs`

### Standard #5: One Function = One Job

**Single responsibility per function**

- Functions <100 lines preferred
- Parameters <5 preferred
- No boolean flags - use separate functions or enums instead
- Function name must clearly describe what it does

### Standard #6: Separate Test Files

**NEVER inline test modules**

- ✅ `file.rs` + `file_test.rs` (side-by-side)
- ❌ `#[cfg(test)] mod tests` inside `file.rs`
- Tests live in same directory as implementation
- Integration tests in `tests/` directory

### Standard #7: No Swiss Army Functions

**No multi-purpose functions**

- No `handle()`, `process()`, `manage()` doing many things
- No `do_thing(x, mode, flag1, flag2)` with complex branching
- Split into focused, single-purpose functions
- Example: Not `validate(value, type, mode)` but `validate_integer_range()`, `validate_string_length()`

### Standard #8: No Generic Names

**Specific, contextual names**

- ✅ `validate_integer_range()`, `TokeniseGblnInput`, `ParseTypeHint`
- ❌ `validate()`, `Processor`, `process()`, `get()`, `handle()`
- Names must convey intent and context

---

## Project Overview

**GBLN (Goblin Bounded Lean Notation)** is the first **LLM-native serialisation format** designed for token efficiency and deterministic parsing. It uses **84% fewer tokens than JSON** in AI contexts whilst providing parse-time type validation through three simple rules.

### Core Value Proposition

- **Type-safe**: Inline type hints with parse-time validation
- **Memory-efficient**: 70% smaller than JSON through bounded types
- **LLM-optimised**: 84% fewer tokens than JSON for AI contexts
- **Human-readable**: Text-based format with clear syntax
- **Git-friendly**: Meaningful diffs, ordered keys preserved
- **Simple parser**: Single-pass, minimal complexity
- **Unique syntax**: No conflicts with existing formats (`:| comments`, `<type>` hints)

---

## Quick Reference

### The Three Rules

GBLN is built on **three simple rules** that enable deterministic parsing and LLM optimisation:

**Rule 1: Structure**
```
record = identifier + value
value  = <type optional> + content
```

**Rule 2: Content Types**
```
(...)  → Single value (type REQUIRED)
{...}  → Object       (type NOT USED)
[...]  → Array        (type OPTIONAL)
```

**Rule 3: Deterministic Parsing**
```
Next char after identifier/<type> determines structure:
  '(' → Single value
  '{' → Object
  '[' → Array
```

### Basic Syntax

**Single Value:**
```gbln
name<s32>(Alice)
```

**Object (Multi-Value):**
```gbln
user{
    id<u32>(12345)           :| 32-bit unsigned integer
    name<s64>(Alice)         :| Max 64 characters
    age<i8>(25)              :| 8-bit signed integer
    active<b>(t)             :| Boolean (t/f)
}
```

**Array (Homogeneous):**
```gbln
tags<s16>[rust python golang]
```

### Type System

| Category | Types | Example |
|----------|-------|---------|
| **Signed Int** | i8, i16, i32, i64 | `age<i8>(25)` |
| **Unsigned Int** | u8, u16, u32, u64 | `id<u32>(12345)` |
| **Float** | f32, f64 | `price<f32>(19.99)` |
| **String** | s2, s4, s8, s16, s32, s64, s128, s256, s512, s1024 | `name<s64>(Alice)` |
| **Boolean** | b | `active<b>(t)` or `active<b>(true)` |
| **Null** | n | `optional<n>()` or `optional<n>(null)` |

### Comments

```gbln
:| This is a comment (unique to GBLN)
```

---

## Project Structure

```
GBLN/
├── .claude/
│   ├── CLAUDE.md           # This file
│   └── Import/             # Original project documentation
│       ├── project-raw.md
│       └── docs/
│           ├── 00-introduction.md
│           ├── 01-index.md
│           ├── 02-specification.md
│           ├── 03-architecture.md
│           └── 04-rust-implementation.md
├── docs/                   # Public documentation (to be created)
├── spec/                   # Formal specification (to be created)
├── core/                   # Core implementations
│   ├── rust/              # Reference implementation (primary)
│   └── c/                 # FFI foundation
├── bindings/              # Language bindings
│   ├── python/
│   ├── javascript/
│   ├── swift/
│   ├── kotlin/
│   ├── go/
│   ├── java/
│   ├── csharp/
│   ├── cpp/
│   ├── ruby/
│   ├── php/
│   ├── perl/
│   └── tcl/
├── tools/                 # CLI tools
│   ├── fmt/              # Formatter
│   ├── lint/             # Linter
│   └── convert/          # JSON↔GBLN converter
├── editors/              # Editor support
│   ├── vscode/
│   ├── vim/
│   ├── zed/
│   ├── jetbrains/
│   └── sublime/
├── benchmarks/           # Performance benchmarks
└── examples/             # Example files
```

---

## Implementation Roadmap

### Current Phase: Core Implementation

**Timeline**: 9 weeks total

#### Week 1-2: Rust Core + C FFI ✅ (Ready to Start)
- [ ] Rust reference parser implementation
- [ ] Type system with all integer/float/string types
- [ ] Parse-time validation engine
- [ ] Comprehensive error messages
- [ ] Test suite (100+ tests)
- [ ] C FFI layer (libgbln.so)

#### Week 3-4: Tier 1 Language Bindings
- [ ] Python (via ctypes)
- [ ] JavaScript/TypeScript (via WASM)
- [ ] Swift (native for iOS/macOS)
- [ ] Kotlin (native for Android/JVM)
- [ ] Go (via CGO)

#### Week 5-6: Tier 2 Language Bindings
- [ ] Java (via JNA)
- [ ] C# (via P/Invoke)
- [ ] C++ (native)

#### Week 7-8: Tier 3 Language Bindings + Tooling
- [ ] Ruby, PHP, Perl, Tcl
- [ ] CLI tools (fmt, lint, validate, convert)
- [ ] Editor plugins (VSCode, Vim, Zed, JetBrains, Sublime)

#### Week 9: Polish & Launch
- [ ] Documentation completion
- [ ] Website + Interactive playground
- [ ] Package manager releases
- [ ] Community launch

---

## Key Technical Details

### Parser Architecture

```
Input String
    ↓
Strip Comments (:|)
    ↓
Tokenize (Lexer)
    ↓
Parse Structure (Recursive Descent)
    ↓
Validate Types (Parse-Time)
    ↓
Build Value Tree
    ↓
Return Result<Value>
```

### Core Rust Types

```rust
pub enum Value {
    I8(i8), I16(i16), I32(i32), I64(i64),
    U8(u8), U16(u16), U32(u32), U64(u64),
    F32(f32), F64(f64),
    Str(String),
    Bool(bool),
    Null,
    Object(HashMap<String, Value>),
    Array(Vec<Value>),
}

pub enum TypeHint {
    I8, I16, I32, I64,
    U8, U16, U32, U64,
    F32, F64,
    Str(usize),  // max characters
    Bool,
    Null,
}
```

### Validation Rules (Parse-Time)

1. **Integer Range**: Values must fit within type's range
   - `age<i8>(999)` → ERROR: 999 out of range [-128, 127]

2. **String Length**: Character count (UTF-8 aware, not bytes)
   - `name<s8>(VeryLongNameHere)` → ERROR: 17 chars > 8

3. **Type Match**: Value must parse as specified type
   - `age<i8>(abc)` → ERROR: "abc" not parseable as integer

4. **Duplicate Keys**: Object keys must be unique
   - Duplicate key detection with line numbers

---

## Special Features & Design Decisions

### 1. No Escaping Needed for Angle Brackets

Angle brackets `<>` in values are automatically handled:

```gbln
html<s256>(<h1>Hello</h1>)
xml<s512>(<user><name>Alice</name></user>)
generic<s64>(Vec<String>)
```

**How**: Parser reads type hint first, then treats all content until matching closing paren as value.

### 2. Nested Parentheses Auto-Handled

```gbln
formula<s64>(f(x) = (x + 1) * (x - 1))
nested<s128>(outer(inner(deepest(x))))
```

**How**: Parser tracks parenthesis depth, only outermost `)` ends value.

### 3. UTF-8 Character Counting

String types count **characters**, not bytes:

```gbln
city<s4>(北京)               :| 2 characters (6 bytes in UTF-8)
emoji<s8>(Hello🔥)           :| 6 characters (5 ASCII + 1 emoji)
```

### 4. Unique Comment Syntax

```gbln
:| This is a comment
```

**Why `:|`**: No other format uses this, prevents conflicts with existing files.

### 5. LLM Token Optimisation

**Development Mode (human-readable):**
```gbln
:| User configuration
user{
    id<u32>(12345)
    name<s64>(Alice)
}
```

**LLM Mode (compressed - 84% fewer tokens):**
```gbln
user{id<u32>(12345)name<s64>(Alice)}
```

**Key Points:**
- Whitespace outside `()` is optional
- Comments can be stripped for production/LLM contexts
- Fully reversible with `gbln fmt`
- Perfect for RAG systems, LLM prompts, fine-tuning data

**Token Savings:**
- JSON: 52,000 tokens (1000 records)
- GBLN compressed: 8,300 tokens (84% reduction)

---

## Common Use Cases

### 1. Configuration Files

```gbln
:| Application Configuration
app{
    name<s32>(My Application)
    version<s16>(1.0.0)
    port<u16>(8080)
    debug<b>(f)
    workers<u8>(4)
}
```

### 2. API Responses

```gbln
response{
    status<u16>(200)
    message<s64>(Success)
    data{
        user{
            id<u32>(12345)
            name<s64>(Alice Johnson)
            role<s16>(admin)
        }
    }
}
```

### 3. IoT Device Communication

```gbln
sensor{
    device_id<s16>(SENS-ENV-001)
    readings{
        temperature<f32>(22.5)
        humidity<u8>(65)
        battery<u8>(87)
    }
}
```

### 4. Database Exports

```gbln
users[
    {id<u32>(1) name<s32>(Alice) age<i8>(25)}
    {id<u32>(2) name<s32>(Bob) age<i8>(30)}
]
```

---

## Development Guidelines

### When Implementing New Features

1. **Reference the Specification**: Always check `.claude/Import/docs/02-specification.md`
2. **Follow Rust Implementation**: Use `.claude/Import/docs/04-rust-implementation.md` as guide
3. **Type-First Approach**: Parse type hint before value, enables validation
4. **Error Messages**: Provide detailed, actionable error messages with suggestions
5. **Test Coverage**: Every feature needs unit + integration tests

### Code Style

**Rust:**
- Use `rustfmt` with default settings
- Follow Rust API guidelines
- Minimize dependencies (no regex, no serde initially)
- Zero-cost abstractions preferred

**Other Languages:**
- Follow language idioms
- Provide type hints where possible (Python, TypeScript)
- Use FFI through C layer (not direct Rust FFI)

### Testing Strategy

```
Unit Tests (60%)       → Individual functions
Integration Tests (30%) → Full parsing workflows
E2E Tests (10%)        → Cross-language compatibility
```

**Test Fixtures**: Shared `.gbln` files in `tests/fixtures/`

---

## Performance Targets

### Memory Usage
- 70% smaller than equivalent JSON
- 40% smaller than Protocol Buffers
- Bounded types prevent buffer overflows

### Parse Speed
- Target: ~65ms for 1000 records
- Trade-off: 30-50% slower than JSON, but type-safe
- Acceptable for most use cases (not ultra-high-throughput)

### Size Comparison (1000 User Records)
- JSON: 156 KB
- Protocol Buffers: 42 KB
- **GBLN: 30 KB** ✨

---

## Common Tasks & Commands

### Working with Documentation

```bash
# Read full specification
cat .claude/Import/docs/02-specification.md

# View architecture
cat .claude/Import/docs/03-architecture.md

# Check implementation guide
cat .claude/Import/docs/04-rust-implementation.md
```

### Starting Rust Implementation

```bash
# Create core Rust project
cd core/rust
cargo new gbln --lib

# Key files to create:
# - src/lib.rs (public API)
# - src/lexer.rs (tokenization)
# - src/parser.rs (parsing logic)
# - src/types.rs (type system)
# - src/value.rs (value representation)
# - src/validator.rs (validation rules)
# - src/error.rs (error handling)
# - src/serialiser.rs (GBLN generation)
```

### Testing

```bash
# Run tests
cargo test

# Run benchmarks
cargo bench

# Test specific module
cargo test lexer

# Generate coverage
cargo tarpaulin
```

---

## Error Message Format

GBLN emphasizes **helpful, actionable error messages**:

```
Error: Integer out of range
  at field: user.age
  value: 999
  type: i8
  valid range: -128 to 127
  line: 5
  column: 15
  
  suggestion: Use i16 or i32 for larger values
```

**Always include:**
- Clear error category
- Field path (for nested structures)
- Expected vs. actual values
- Line and column numbers
- Helpful suggestion for fixing

---

## EBNF Grammar Reference

```ebnf
document     = value ;
value        = object | array | primitive ;

object       = "{" , [ key_values ] , "}" ;
key_values   = key_value , { whitespace , key_value } ;

key_value    = key , "<" , type_hint , ">" , "(" , raw_value , ")" ;
key          = identifier ;

array        = "[" , [ array_items ] , "]" ;
array_items  = typed_array | mixed_array | object_array ;

typed_array  = key , "<" , type_hint , ">" , "[" , [ values ] , "]" ;
mixed_array  = typed_value , { whitespace , typed_value } ;
object_array = object , { whitespace , object } ;

typed_value  = "<" , type_hint , ">" , "(" , raw_value , ")" ;
raw_value    = { char - ")" | escaped_char } ;

type_hint    = type_base , [ size ] ;
type_base    = "s" | "i" | "u" | "f" | "b" | "n" ;

identifier   = ( letter | "_" ) , { letter | digit | "_" } ;
```

---

## FAQ for Development

### Q: Why Rust as reference implementation?

**A**: Memory safety, performance, zero-cost abstractions, and excellent WebAssembly support. Can compile to C FFI for other languages.

### Q: Why C FFI layer instead of direct Rust FFI?

**A**: C ABI is stable and universal. Every language can bind to C. Rust FFI is less portable.

### Q: What about binary format?

**A**: Future consideration. Text format is priority for human-readability and Git-friendliness.

### Q: How to handle very large files?

**A**: Implement streaming parser (`StreamParser<R: Read>`) for memory-efficient processing.

### Q: What about schema validation?

**A**: Optional feature for Phase 2. Types in GBLN provide basic schema. External schema files can add constraints (enum values, regex patterns, etc.).

### Q: Backward compatibility strategy?

**A**: Semantic versioning (SemVer). Specification version embedded in parser. Breaking changes only in major versions.

---

## Integration Examples

### Rust

```rust
use gbln::{parse, Value};

let input = r#"
    user{
        id<u32>(12345)
        name<s64>(Alice)
    }
"#;

let value = parse(input)?;
if let Some(id) = value["user"]["id"].as_u32() {
    println!("User ID: {}", id);
}
```

### Python

```python
import gbln

data = gbln.parse("""
    user{
        id<u32>(12345)
        name<s64>(Alice)
    }
""")

print(data["user"]["id"])  # 12345
```

### JavaScript

```javascript
import { parse } from 'gbln';

const data = parse(`
    user{
        id<u32>(12345)
        name<s64>(Alice)
    }
`);

console.log(data.user.id);  // 12345
```

---

## Resources

### Internal Documentation
- **Full Spec**: `.claude/Import/docs/02-specification.md`
- **Architecture**: `.claude/Import/docs/03-architecture.md`
- **Rust Guide**: `.claude/Import/docs/04-rust-implementation.md`
- **Original Concept**: `.claude/Import/project-raw.md`

### External Resources (Planned)
- Website: `gbln.dev` (coming soon)
- GitHub: `github.com/gbln` (coming soon)
- Discord: Community server (coming soon)

---

## Project Philosophy

### Design Principles

1. **Type Safety First**: Prevent errors at parse-time, not runtime
2. **Human-Friendly**: Readable by developers, editable with any text editor
3. **Memory-Conscious**: Bounded types prevent waste and vulnerabilities
4. **Simple Complexity**: Complex enough to be useful, simple enough to implement
5. **Git-Optimized**: Meaningful diffs, no binary blobs

### Non-Goals

- ❌ Replace Protocol Buffers for ultra-high-throughput systems
- ❌ Support for streaming large arrays (files > 100MB)
- ❌ Schema evolution like Protobuf (use versioning instead)
- ❌ Compression (use gzip/zstd at transport layer)
- ❌ Cryptographic signing (use separate tools)

---

## Contributors

**Project Lead**: Vivian Voss (ask@vvoss.dev)

**License**: 
- Specification: CC0 (Public Domain)
- Implementation: MIT License

---

## Version History

### v1.1 (2025-01-20)
- Added s2, s4, s512, s1024 string types
- Clarified UTF-8 character counting
- Added duplicate key validation
- Improved error message format

### v1.0 (2025-01-15)
- Initial specification release

---

## Next Steps for Claude

When assisting with this project:

1. **Always refer to this guide** for project context
2. **Check specification** before implementing features
3. **Follow Rust-first approach** for reference implementation
4. **Emphasize type safety** in all code
5. **Provide detailed error messages** with suggestions
6. **Write tests** for every feature
7. **Keep code simple** - avoid over-engineering
8. **Document decisions** - explain "why", not just "what"

---

*GBLN - Goblin Bounded Lean Notation*  
*Type-safe data that speaks clearly* 🦇

---

**End of Guide**
