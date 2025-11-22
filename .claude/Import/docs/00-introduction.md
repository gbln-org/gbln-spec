# GBLN Project - Introduction

## Project Overview

**GBLN (Goblin Bounded Lean Notation)** is a type-safe, memory-efficient serialization format that validates data at parse-time. This project implements parsers and libraries for 13 programming languages.

## Vision

Create a universal data format that combines:
- Human-readability of JSON
- Type-safety of Protocol Buffers
- Memory-efficiency through bounded types
- No separate schema files required
- Parse-time validation preventing runtime errors

## Key Differentiators

### vs JSON
- ✅ 70% smaller memory footprint
- ✅ Parse-time type validation
- ✅ Bounded types prevent buffer overflows
- ✅ Comments supported

### vs Protocol Buffers
- ✅ No separate schema files
- ✅ Human-readable text format
- ✅ Git-friendly diffs
- ✅ No code generation needed

### vs YAML
- ✅ Type-safe with validation
- ✅ Simpler syntax (no anchors/aliases)
- ✅ Faster parsing
- ✅ No indentation ambiguity

### vs TOON
- ✅ Stronger type-safety (i8, u32, s64 vs generic types)
- ✅ 85% less memory usage at runtime
- ✅ Parse-time validation (not runtime)
- ✅ Buffer overflow prevention
- ✅ Mobile-first (Swift + Kotlin support)

## Target Languages

### Must-Have (Phase 1-2)
1. **Rust** - Reference implementation + WASM
2. **C** - Universal FFI foundation
3. **Python** - Data science ecosystem
4. **JavaScript/TypeScript** - Web/Node.js
5. **Swift** - iOS/macOS/watchOS/tvOS
6. **Kotlin** - Android/JVM/Multiplatform
7. **Go** - Cloud/DevOps

### Should-Have (Phase 3)
8. **Java** - Enterprise applications
9. **C#** - .NET ecosystem
10. **C++** - Performance-critical applications

### Nice-to-Have (Phase 4)
11. **Ruby** - Web frameworks
12. **PHP** - WordPress/Drupal
13. **Perl** - Legacy systems
14. **Tcl** - Embedded/Automation

## Core Features

### Type System
- **Integers**: i8, i16, i32, i64 (signed) / u8, u16, u32, u64 (unsigned)
- **Floats**: f32, f64
- **Strings**: s8, s16, s32, s64, s128, s256 (character count bounded)
- **Boolean**: b (t/f, true/false, 0/1)
- **Null**: n

### Data Structures
- Objects: `user{id<u32>(123) name<s32>(Alice)}`
- Homogeneous Arrays: `numbers<i32>[1 2 3]`
- Mixed Arrays: `mixed[<i32>(42) <s16>(hello)]`
- Nested structures with unlimited depth

### Validation
- Integer range checking at parse-time
- String length validation (character count)
- Type matching validation
- Detailed error messages with suggestions

### Special Features
- Comments: `:| This is a comment`
- No escaping needed for angle brackets in values
- Automatic nested parentheses handling
- UTF-8 aware string length counting

## Project Goals

1. **Universal Adoption**: Support 13+ languages day-1
2. **Production Ready**: Comprehensive tests, benchmarks, documentation
3. **Mobile First**: Native iOS (Swift) and Android (Kotlin) support
4. **Developer Experience**: Clear errors, easy integration, good docs
5. **Performance**: Fast parsing, minimal memory usage
6. **Safety**: Prevent buffer overflows, type errors, data corruption

## Success Metrics

- [ ] 13 language implementations complete
- [ ] 100% test coverage for core parser
- [ ] Benchmarks showing 70%+ memory savings vs JSON
- [ ] Documentation for all languages
- [ ] Package managers: cargo, npm, pip, maven, nuget, etc.
- [ ] Editor support: VSCode, Vim, Sublime
- [ ] CLI tools: formatter, linter, validator, converter
- [ ] Website with interactive playground
- [ ] Community: GitHub stars, Discord server

## Timeline

**Total Duration**: 9 weeks

- **Weeks 1-2**: Rust core + C FFI
- **Weeks 3-4**: Python, JS, Swift, Kotlin, Go
- **Weeks 5-6**: Java, C#, C++
- **Weeks 7-8**: Ruby, PHP, Perl, Tcl
- **Week 9**: Testing, docs, tooling, launch prep

## Repository Structure

```
gbln/
├── docs/               # Documentation
├── spec/               # GBLN specification
├── core/               # Core implementations
│   ├── rust/          # Reference implementation
│   └── c/             # FFI foundation
├── bindings/          # Language bindings
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
├── tools/             # CLI tools
│   ├── fmt/          # Formatter
│   ├── lint/         # Linter
│   └── convert/      # JSON↔GBLN converter
├── editors/          # Editor support
│   ├── vscode/
│   ├── vim/
│   ├── zed/
│   ├── jetbrains/
│   └── sublime/
├── benchmarks/       # Performance benchmarks
└── examples/         # Example files
```

## Getting Started

See `01-index.md` for the complete documentation index.

## License

- **Specification**: CC0 (Public Domain)
- **Reference Implementation**: MIT License
- **Language Bindings**: MIT License

## Author

Vivian Voss (ask@vvoss.dev)

---

*GBLN - Type-safe data that speaks clearly*
