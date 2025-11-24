<div align="center">

<img src="logo.svg" alt="GBLN Logo" width="120" height="120">

# GBLN - Goblin Bounded Lean Notation

**The first type-safe LLM-native data format: Deterministic, Token-Efficient, Memory-Bounded**

[![Status](https://img.shields.io/badge/status-specification-blue)](docs/01-specification.md)
[![Version](https://img.shields.io/badge/version-1.0-green)](docs/00-introduction.md)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue)](LICENSE)

</div>

---

## 🗂️ Project Components

This repository serves as the central hub for the GBLN ecosystem. All implementation repositories are included as Git submodules.

### Core Implementation
- **[📋 Specification](docs/)** - The GBLN format specification (this repo)
- [![Rust](https://img.shields.io/badge/Rust-v0.1.0-blue?logo=rust&logoColor=white)](https://github.com/gbln-org/gbln-rust) **[Rust Core](https://github.com/gbln-org/gbln-rust)** - Reference implementation and parser library
- [![C FFI](https://img.shields.io/badge/C%20FFI-v0.1.0-blue?logo=c&logoColor=white)](https://github.com/gbln-org/gbln-ffi) **[C FFI](https://github.com/gbln-org/gbln-ffi)** - C interface layer for language bindings
- <span style="filter: grayscale(100%); opacity: 0.6;">[![CLI](https://img.shields.io/badge/CLI-planned-red?logo=gnubash&logoColor=white)](https://github.com/gbln-org/gbln-tools)</span> **[CLI Tools](https://github.com/gbln-org/gbln-tools)** - Command-line tools (fmt, validate, convert)

### Language Bindings
- <span style="filter: grayscale(100%); opacity: 0.6;">[![Python](https://img.shields.io/badge/Python-planned-red?logo=python&logoColor=white)](https://github.com/gbln-org/gbln-python)</span> **[Python](https://github.com/gbln-org/gbln-python)** - Python bindings
- <span style="filter: grayscale(100%); opacity: 0.6;">[![JavaScript](https://img.shields.io/badge/JavaScript-planned-red?logo=javascript&logoColor=white)](https://github.com/gbln-org/gbln-javascript)</span> **[JavaScript/TS](https://github.com/gbln-org/gbln-javascript)** - WASM bindings
- <span style="filter: grayscale(100%); opacity: 0.6;">[![Swift](https://img.shields.io/badge/Swift-planned-red?logo=swift&logoColor=white)](https://github.com/gbln-org/gbln-swift)</span> **[Swift](https://github.com/gbln-org/gbln-swift)** - iOS/macOS bindings
- <span style="filter: grayscale(100%); opacity: 0.6;">[![Kotlin](https://img.shields.io/badge/Kotlin-planned-red?logo=kotlin&logoColor=white)](https://github.com/gbln-org/gbln-kotlin)</span> **[Kotlin](https://github.com/gbln-org/gbln-kotlin)** - Android/JVM bindings
- <span style="filter: grayscale(100%); opacity: 0.6;">[![Go](https://img.shields.io/badge/Go-planned-red?logo=go&logoColor=white)](https://github.com/gbln-org/gbln-go)</span> **[Go](https://github.com/gbln-org/gbln-go)** - Go bindings
- <span style="filter: grayscale(100%); opacity: 0.6;">[![Java](https://img.shields.io/badge/Java-planned-red?logo=openjdk&logoColor=white)](https://github.com/gbln-org/gbln-java)</span> **[Java](https://github.com/gbln-org/gbln-java)** - Java bindings
- <span style="filter: grayscale(100%); opacity: 0.6;">[![Ruby](https://img.shields.io/badge/Ruby-planned-red?logo=ruby&logoColor=white)](https://github.com/gbln-org/gbln-ruby)</span> **[Ruby](https://github.com/gbln-org/gbln-ruby)** - Ruby bindings
- <span style="filter: grayscale(100%); opacity: 0.6;">[![PHP](https://img.shields.io/badge/PHP-planned-red?logo=php&logoColor=white)](https://github.com/gbln-org/gbln-php)</span> **[PHP](https://github.com/gbln-org/gbln-php)** - PHP bindings
- <span style="filter: grayscale(100%); opacity: 0.6;">[![Perl](https://img.shields.io/badge/Perl-planned-red?logo=perl&logoColor=white)](https://github.com/gbln-org/gbln-perl)</span> **[Perl](https://github.com/gbln-org/gbln-perl)** - Perl bindings
- <span style="filter: grayscale(100%); opacity: 0.6;">[![Tcl](https://img.shields.io/badge/Tcl-planned-red?logo=tcl&logoColor=white)](https://github.com/gbln-org/gbln-tcl)</span> **[Tcl](https://github.com/gbln-org/gbln-tcl)** - Tcl bindings
- <span style="filter: grayscale(100%); opacity: 0.6;">[![C#](https://img.shields.io/badge/C%23-planned-red?logo=csharp&logoColor=white)](https://github.com/gbln-org/gbln-csharp)</span> **[C#](https://github.com/gbln-org/gbln-csharp)** - .NET bindings
- <span style="filter: grayscale(100%); opacity: 0.6;">[![C++](https://img.shields.io/badge/C++-planned-red?logo=cplusplus&logoColor=white)](https://github.com/gbln-org/gbln-cpp)</span> **[C++](https://github.com/gbln-org/gbln-cpp)** - C++ bindings

### Editor Support
- <span style="filter: grayscale(100%); opacity: 0.6;">[![VSCode](https://img.shields.io/badge/VSCode-planned-red?logo=visualstudiocode&logoColor=white)](https://github.com/gbln-org/gbln-vscode)</span> **[VSCode](https://github.com/gbln-org/gbln-vscode)** - VS Code extension
- <span style="filter: grayscale(100%); opacity: 0.6;">[![Vim](https://img.shields.io/badge/Vim-planned-red?logo=vim&logoColor=white)](https://github.com/gbln-org/gbln-vim)</span> **[Vim](https://github.com/gbln-org/gbln-vim)** - Vim/Neovim plugin
- <span style="filter: grayscale(100%); opacity: 0.6;">[![JetBrains](https://img.shields.io/badge/JetBrains-planned-red?logo=jetbrains&logoColor=white)](https://github.com/gbln-org/gbln-jetbrains)</span> **[JetBrains](https://github.com/gbln-org/gbln-jetbrains)** - IntelliJ, PyCharm, etc.
- <span style="filter: grayscale(100%); opacity: 0.6;">[![Sublime](https://img.shields.io/badge/Sublime-planned-red?logo=sublimetext&logoColor=white)](https://github.com/gbln-org/gbln-sublime)</span> **[Sublime](https://github.com/gbln-org/gbln-sublime)** - Sublime Text plugin
- <span style="filter: grayscale(100%); opacity: 0.6;">[![Zed](https://img.shields.io/badge/Zed-planned-red?logo=zedindustries&logoColor=white)](https://github.com/gbln-org/gbln-zed)</span> **[Zed](https://github.com/gbln-org/gbln-zed)** - Zed editor extension

### Website
- <span style="filter: grayscale(100%); opacity: 0.6;">[![Website](https://img.shields.io/badge/gbln.dev-planned-red?logo=html5&logoColor=white)](https://github.com/gbln-org/gbln-website)</span> **[gbln.dev](https://github.com/gbln-org/gbln-website)** - Official website & playground

### Getting Started

```bash
# Clone only the specification
git clone https://github.com/gbln-org/gbln.git

# Clone with all implementation repositories
git clone --recursive https://github.com/gbln-org/gbln.git

# Clone a specific implementation
git clone https://github.com/gbln-org/gbln-rust.git
```

---

## 🎯 The Problem

Text-based data formats weren't designed for the AI era:

- **JSON**: No types, wasteful tokens, no validation
- **YAML**: Ambiguous, slow, error-prone
- **TOML**: Limited nesting, verbose, no type safety

**GBLN solves this.**

---

## ⚡ The Solution

**GBLN uses a dual-file system:**

**Source File (`.gbln`)** - Human-Editable:
```gbln
:| config.gbln - For Developers & Git
server{
  host<s64>(api.example.com)
  port<u16>(8080)
  workers<u8>(4)
}
```

**I/O Format (`.io.gbln.xz`)** - Optimised:
```gbln
server{host<s64>(api.example.com)port<u16>(8080)workers<u8>(4)}
```
*(Then XZ compressed to binary)*

**Workflow:**
```bash
# 1. Edit human-readable
vim config.gbln

# 2. Generate I/O format
gbln write config.gbln  # → config.io.gbln.xz (73% smaller)

# 3. App uses I/O format
myapp --config config.io.gbln.xz
```

---

## 🚀 Key Features

### 1. Prevent Errors Before They Happen
- **Parse-time validation** catches bad data at the gate
- Type errors rejected **before** entering your system
- Catch **LLM hallucinations** immediately
- **No runtime surprises** from invalid data

### 2. Token Efficient by Design
- **86% fewer tokens** than JSON for AI contexts (real benchmark)
- **Deterministic structure** guides AI code generation
- Whitespace compression built-in
- Perfect for **RAG systems**, **prompts**, **fine-tuning**

### 3. Rock Solid Memory Prediction
- **Bounded types** mean predictable allocation
- `s32` = max 32 characters, `u16` = 0-65535, known at parse-time
- No buffer overflows, no OOM surprises
- Perfect for **IoT, mobile, embedded**

### 4. KISS Syntax Rules
- **Three simple rules**, zero ambiguity
- **O(1) lookahead** - next character determines structure
- Predictable for **humans AND machines**
- Easy to implement parsers

### 5. Progressive Complexity
- **Types optional** - start fast like JSON
- **Add types incrementally** - no big rewrite needed
- Mix typed and untyped in same file
- **Unique**: JSON has no types, Protobuf/TOON force types everywhere

---

## 📊 Comparison - Real Benchmarks

### 100 Employees (Nested Data, Production-Ready)

**I/O Formats** (Wire/Storage/LLM):

| Format | Bytes | vs JSON Minified | Type-Safe | Memory-Bounded |
|--------|-------|------------------|-----------|----------------|
| **GBLN I/O (`.io.gbln.xz`)** | **~16,500** ⭐⭐⭐ | **67% smaller** | ✅ ⭐ | ✅ ⭐ |
| **GBLN MINI (`.io.gbln`)** | **49,276** ⭐⭐ | **0.4% smaller** | ✅ ⭐ | ✅ ⭐ |
| JSON (minified) | 49,466 | baseline | ❌ | ❌ |
| TOON | 53,601 | 8.4% larger | ❌ | ❌ |
| YAML | 57,701 | 16.6% larger | ❌ | ❌ |

**Source Formats** (Development, Git-Tracked):

| Format | Bytes | vs JSON Pretty | Type-Safe | Memory-Bounded |
|--------|-------|----------------|-----------|----------------|
| TOON | 53,601 | -32% | ❌ | ❌ |
| YAML | 57,701 | -27% | ❌ | ❌ |
| **GBLN (`.gbln`)** | **66,273** | **-17%** | ✅ ⭐ | ✅ ⭐ |
| JSON (indent=2) | 79,394 | baseline | ❌ | ❌ |

**The Real Win**: 
- GBLN MINI is **0.4% smaller** than JSON minified (~190 bytes) **AND includes type safety + memory bounds**
- GBLN I/O is **67% smaller** than JSON minified with XZ compression
- All GBLN variants parse to same validated data structure

### Key Insights

**File Sizes** (100 employee records):
- `.gbln` (source): 66,273 bytes (17% smaller than JSON pretty, Git-tracked)
- `.io.gbln` (MINI): 49,276 bytes (0.4% smaller than JSON minified)
- `.io.gbln.xz` (I/O): ~16,500 bytes (67% smaller than JSON minified)

**But GBLN adds** (at zero size cost):
- ✅ **Parse-time type validation** - Catch errors before runtime
- ✅ **Bounded strings** - `s32` = max 32 chars, no buffer overflows
- ✅ **Memory predictability** - `u32` = 0-4B range, known at parse-time
- ✅ **LLM validation** - Reject hallucinated data immediately
- ✅ **Deterministic parsing** - 3 simple rules, O(1) lookahead
- ✅ **Progressive complexity** - Types optional for prototyping

**What is "GBLN"?**
- **GBLN** = The format (no whitespace except in values)
- **GBLN formatted** = Pretty-printed for humans (development only)
- File extensions: `*.gbln` (production) or `*.pretty.gbln` (formatted)
- Wire/cache/LLM always use plain GBLN (compact)

**When Each Format Wins**:
- **GBLN**: Type safety + predictable memory + compact (this benchmark) ⭐
- **TOON**: Flat/uniform tables (CSV-style, not tested here)
- **JSON**: Maximum ecosystem compatibility
- **YAML**: Human-editable configs (with gotchas)

**Honest Assessment**: GBLN and JSON minified are **virtually identical in size**. GBLN's advantage is **reliability, not bytes saved**.

---

## 🎓 The Three Rules

GBLN is built on **three simple rules** that enable deterministic parsing and LLM optimisation:

### Rule 1: Structure
```
record = identifier + value
value  = <type optional> + content
```

### Rule 2: Content Types
```
(...)  → Single value (type REQUIRED)
{...}  → Object       (type NOT USED)
[...]  → Array        (type OPTIONAL)
```

### Rule 3: Deterministic Parsing
```
Next char after identifier/<type> determines structure:
  '(' → Single value
  '{' → Object
  '[' → Array
```

**That's it. No special cases, no ambiguity.**

---

## 📖 Examples

**Note**: All examples below show the **formatted version** (with whitespace for human readability). In production, GBLN uses the compact format without structural whitespace.

### Configuration File
```gbln
:| Application Configuration (formatted for humans)
app{
    name<s32>(My Application)
    version<s16>(1.0.0)
    port<u16>(8080)
    workers<u8>(4)
    debug<b>(f)
}
```

**Production format** (what actually gets sent/stored):
```gbln
app{name<s32>(My Application)version<s16>(1.0.0)port<u16>(8080)workers<u8>(4)debug<b>(f)}
```

---

### API Response
```gbln
:| Formatted for development/debugging
response{
    status<u16>(200)
    message<s64>(Success)
    data{
        user{
            id<u32>(12345)
            name<s64>(Alice Johnson)
            email<s64>(alice@example.com)
        }
    }
}
```

**Production format**:
```gbln
response{status<u16>(200)message<s64>(Success)data{user{id<u32>(12345)name<s64>(Alice Johnson)email<s64>(alice@example.com)}}}
```

---

### IoT Sensor Data
```gbln
:| Formatted for development
sensor{
    device_id<s16>(SENS-001)
    temperature<f32>(22.5)
    humidity<u8>(65)
    battery<u8>(87)
}
```

**Production format** (actual GBLN over wire/cache):
```gbln
sensor{device_id<s16>(SENS-001)temperature<f32>(22.5)humidity<u8>(65)battery<u8>(87)}
```

---

**Key Takeaway**: GBLN is always compact. Formatting with indentation is optional and only for developers editing files. The `gbln write` tool generates optimized I/O files, while `gbln read` uses smart lookup to find the best available format.

---

## 🛠️ Implementation Status

| Component | Status |
|-----------|--------|
| **Specification** | ✅ Complete (v1.0) |
| **Rust Core** | 🚧 In Progress |
| **C FFI** | 📋 Planned |
| **Python** | 📋 Planned |
| **JavaScript/TypeScript** | 📋 Planned |
| **Swift** | 📋 Planned |
| **Kotlin** | 📋 Planned |
| **Go** | 📋 Planned |
| **CLI Tools** | 📋 Planned |
| **Editor Support** | 📋 Planned |

---

## 📚 Documentation

### Core Documents
- **[00-introduction.md](docs/00-introduction.md)** - Overview and motivation
- **[01-specification.md](docs/01-specification.md)** - Complete formal specification
- **[02-examples.md](docs/02-examples.md)** - Comprehensive examples
- **[03-comparison.md](docs/03-comparison.md)** - vs JSON, YAML, TOML
- **[04-llm-optimisation.md](docs/04-llm-optimisation.md)** - LLM token efficiency guide

### Quick Links
- [Type System Reference](docs/01-specification.md#type-system)
- [Grammar (EBNF)](docs/01-specification.md#grammar-ebnf)
- [Validation Rules](docs/01-specification.md#validation-rules)
- [Best Practises](docs/02-examples.md#best-practises)
- [Migration Guides](docs/03-comparison.md#migration-guides)

---

## 🎯 Use Cases

### ✅ Perfect For

- **LLM Contexts** - RAG systems, prompts, fine-tuning
- **Configuration Files** - Type-safe, validated configs
- **API Responses** - Compact, type-safe data exchange
- **IoT Communication** - Memory-efficient, bounded types
- **Mobile Apps** - Low bandwidth, type validation
- **Database Exports** - Type-preserved, human-readable

### ⚠️ Consider Carefully

- **Ultra-high-throughput** (>1M ops/sec) - Protocol Buffers may be faster
- **Very large files** (>100MB) - Streaming parsers needed
- **Legacy systems** - If they only speak JSON/XML

---

## 🚀 Getting Started

### Read the Specification
```bash
# Start with the introduction
cat docs/00-introduction.md

# Complete specification
cat docs/01-specification.md
```

### Try Examples
```bash
# See comprehensive examples
cat docs/02-examples.md
```

### Understand LLM Benefits
```bash
# Token efficiency analysis
cat docs/04-llm-optimisation.md
```

### (Coming Soon) Install Parser
```bash
# Rust
cargo add gbln

# Python
pip install gbln

# JavaScript
npm install gbln
```

---

## 🤝 Contributing

GBLN is open-source and welcomes contributions:

- **Specification Feedback**: Review and suggest improvements
- **Parser Implementation**: Implement in your favourite language
- **Tooling**: Build formatters, linters, converters
- **Editor Support**: Create plugins for your editor
- **Documentation**: Improve guides and examples

**See:** [CONTRIBUTING.md](CONTRIBUTING.md) *(coming soon)*

---

## 📜 License

**Apache License 2.0**

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

---

## 📞 Contact

**Project Lead**: Vivian Voss  
**Email**: ask+gbln@vvoss.dev  
**Website**: gbln.dev *(coming soon)*  
**GitHub**: github.com/gbln *(coming soon)*

---

## 🎉 Why GBLN?

> **"We built GBLN because existing formats weren't designed for the AI era. JSON wastes tokens, YAML is ambiguous, and Protocol Buffers require schemas. GBLN gives you type safety, token efficiency, and deterministic parsing in one elegant format."**

**Three rules. Zero ambiguity. 84% fewer tokens.**

---

<div align="center">

<img src="logo.svg" alt="GBLN" width="80" height="80">

*GBLN - Goblin Bounded Lean Notation*  
*The first type-safe, memory-bounded LLM-native data format*

**Built for the AI era. Optimised for machines. Readable by humans.**

</div>
