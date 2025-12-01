# GBLN Language Bindings - Quality Standard Notes

**Version**: 1.0  
**Date**: 2025-11-29  
**Purpose**: Quality checkup findings and standards for consistent binding development

---

## Executive Summary

This document establishes quality standards for GBLN language bindings based on analysis of completed implementations (Python, Kotlin, Swift, JavaScript).

**Key Finding**: We need **TWO distinct binding categories** with different feature sets:

1. **FS-Enabled Bindings** (Python, Swift, Kotlin, Go, etc.)
   - Full feature set: parse, serialise, **io** (file read/write)
   - Uses system filesystem APIs
   
2. **FS-Disabled Bindings** (JavaScript/Browser, WASM-only)
   - Limited feature set: parse, serialise only
   - No io module (browser security restrictions)

---

## Binding Categories

### Category 1: FS-Enabled Bindings (Full Feature Set)

**Languages**: Python, Swift, Kotlin, Go, Java, C#, C++, Ruby, PHP, Perl, Tcl

**Mandatory Modules**:
- ✅ **FFI/Wrapper** - Platform detection, library loading, memory management
- ✅ **Parser** - `parse()`, `parseFile()` 
- ✅ **Serialiser** - `toString()`, `toStringPretty()`
- ✅ **Value** - Bidirectional conversion (language types ↔ GBLN)
- ✅ **IO** - `readFile()`, `writeFile()` with filesystem access
- ✅ **Config** - Configuration dataclass/struct
- ✅ **Error** - Custom exception/error types

**File Structure Pattern** (using Python as template):
```
bindings/{language}/
├── src/{package}/
│   ├── ffi.{ext}           # FFI layer (400-500 lines max)
│   ├── value.{ext}         # Value conversion (300-400 lines max)
│   ├── parse.{ext}         # Parser API (100-150 lines max)
│   ├── serialise.{ext}     # Serialiser API (100-150 lines max)
│   ├── io.{ext}            # ⚠️ MANDATORY for FS-enabled (100-200 lines max)
│   ├── config.{ext}        # Configuration (50-100 lines max)
│   └── error.{ext}         # Error types (50-100 lines max)
├── tests/
│   ├── test_parse.{ext}    # Parser tests
│   ├── test_serialise.{ext}# Serialiser tests
│   ├── test_io.{ext}       # ⚠️ IO tests (file read/write)
│   ├── test_objects.{ext}  # Object/array tests
│   └── test_ffi.{ext}      # FFI tests
├── README.md               # Usage documentation
├── LICENSE                 # MIT License
└── {build-file}            # setup.py, build.gradle.kts, Package.swift, etc.
```

### Category 2: FS-Disabled Bindings (Limited Feature Set)

**Languages**: JavaScript (browser), TypeScript (browser), any WASM-only binding

**Mandatory Modules**:
- ✅ **Parser** - `parse()` only (string input)
- ✅ **Serialiser** - `toString()`, `toStringPretty()`
- ✅ **Value** - Bidirectional conversion
- ✅ **Error** - Custom error types
- ❌ **NO IO module** - Browser security prevents filesystem access
- ❌ **NO Config** (optional, only if needed for serialiser options)

**File Structure Pattern**:
```
bindings/{language}/
├── src/
│   ├── parser.{ext}        # Parser API (string only)
│   ├── serialiser.{ext}    # Serialiser API
│   ├── value.{ext}         # Value conversion
│   └── error.{ext}         # Error types
├── tests/
│   ├── parse.test.{ext}
│   └── serialise.test.{ext}
├── README.md
└── {build-file}
```

---

## Quality Checkup Findings

### ✅ Python Bindings (FS-Enabled Reference)

**Status**: Production-ready, reference implementation

**Metrics**:
- Files: 8 source files + 4 test files
- Lines: ffi.py (506), value.py (385), io.py (150), parse.py (125), serialise.py (102)
- Tests: 44 tests passing, 75% coverage
- Documentation: Complete README, type hints (py.typed)

**Strengths**:
- ✅ Complete feature set (parse, serialise, io)
- ✅ Excellent file size discipline (all <400 lines except ffi.py at 506)
- ✅ Platform detection working (macOS/Linux/Windows/FreeBSD)
- ✅ Memory management via weakref.finalize()
- ✅ Comprehensive test suite with fixtures
- ✅ Docker-based multi-platform testing

**Reference for**: All FS-enabled bindings should follow this structure

### ⚠️ Kotlin Bindings (FS-Enabled, JNA)

**Status**: Incomplete - missing io module

**Metrics**:
- Files: 4 source files + 1 test file
- Tests: 2 tests passing (library loads, basic parse)
- Documentation: build.gradle.kts present

**Issues**:
1. ❌ **CRITICAL: Missing io.kt module** - FS-enabled binding MUST have file I/O
2. ❌ **Incomplete test coverage** - Only 2 tests, no io tests, no object/array tests
3. ❌ **Missing Config.kt** - Should have GblnConfig for consistency
4. ❌ **Missing Error.kt** - Only using generic GblnError
5. ⚠️ **Object parsing limitation** - Returns empty {} (C FFI limitation, documented)

**Required Actions**:
1. ✅ Add `io.kt` with `readFile()`, `writeFile()` functions
2. ✅ Add `Config.kt` with GblnConfig data class
3. ✅ Add `Error.kt` with specific error types (ParseError, SerialiseError, IoError)
4. ✅ Add comprehensive test suite (test_io.kt, test_objects.kt, test_serialise.kt)
5. ✅ Add fixtures/ directory with test .gbln files

**Template**: Should match Python structure exactly (adapted to Kotlin idioms)

### ✅ Swift Bindings (FS-Enabled)

**Status**: Complete

**Metrics**:
- Files: 7 source files + 5 test files
- Structure: Sources/GBLN/*.swift + Tests/GBLNTests/*.swift
- Tests: Multiple test files (ParserTests, SerialiserTests, IOTests, ValueConversionTests, RoundtripTests)

**Strengths**:
- ✅ Complete feature set (parse, serialise, io)
- ✅ Comprehensive test suite
- ✅ IO module present (IO.swift, IOTests.swift)
- ✅ Config and Error modules present
- ✅ Swift Package Manager integration

**Reference for**: Good example of FS-enabled binding

### ✅ JavaScript Bindings (FS-Disabled Reference)

**Status**: Complete for FS-disabled scenario

**Metrics**:
- Implementation: WASM-based (wasm-pack)
- Structure: pkg/nodejs/ with WASM bindings
- Tests: Basic test suite present

**Strengths**:
- ✅ Correctly omits io module (browser limitation)
- ✅ WASM approach appropriate for JavaScript
- ✅ Parse and serialise working

**Reference for**: All FS-disabled bindings (browser-based)

**Note**: Node.js *could* have io module (fs API available), but current WASM approach doesn't support it. This is acceptable for browser-first design.

---

## Mandatory Quality Standards

### File Size Discipline (KISS Principle)

**Rule**: No file >400 lines (except FFI wrapper, max 500 lines)

**Enforcement**:
```bash
# Check before commit
find src -name "*.{py,kt,swift,go}" -exec wc -l {} \; | awk '$1 > 400'
```

**Rationale**: Forces single-responsibility, prevents monolithic files

### Module Requirements by Category

#### FS-Enabled Bindings (MANDATORY)

| Module | Lines | Purpose | Status Check |
|--------|-------|---------|--------------|
| ffi/wrapper | 400-500 | Platform detection, library loading | `grep "def _find_library\|fun findLibrary\|func findLibrary"` |
| value | 300-400 | Bidirectional conversion | `grep "def.*to_python\|fun.*toKotlin\|func.*toSwift"` |
| parse | 100-150 | Parser API | `grep "def parse\|fun parse\|func parse"` |
| serialise | 100-150 | Serialiser API | `grep "def to_string\|fun toString\|func toString"` |
| **io** | **100-200** | **File I/O (MANDATORY)** | `grep "def.*file\|fun.*File\|func.*file"` |
| config | 50-100 | Configuration | `grep "class.*Config\|data class.*Config\|struct.*Config"` |
| error | 50-100 | Error types | `grep "class.*Error\|sealed class\|enum.*Error"` |

#### FS-Disabled Bindings (MANDATORY)

| Module | Lines | Purpose | Status Check |
|--------|-------|---------|--------------|
| parse | 100-150 | Parser API (string only) | `grep "function parse\|def parse"` |
| serialise | 100-150 | Serialiser API | `grep "function toString\|def toString"` |
| value | 200-300 | Bidirectional conversion | `grep "toJS\|fromJS\|toWasm"` |
| error | 50-100 | Error types | `grep "class.*Error\|Error extends"` |

### Test Coverage Requirements

**Minimum**: 70% code coverage, all modules tested

**Required Test Files** (FS-Enabled):
- ✅ `test_parse` - Parse primitives, objects, arrays
- ✅ `test_serialise` - Serialise to string, pretty
- ✅ `test_io` - File read/write
- ✅ `test_objects` - Object iteration, nested structures
- ✅ `test_ffi` - Library loading, platform detection
- ✅ `test_config` - Configuration options

**Required Test Files** (FS-Disabled):
- ✅ `test_parse` - Parse primitives, objects, arrays
- ✅ `test_serialise` - Serialise to string

### FFI Layer Requirements

**Mandatory Features**:
1. ✅ Platform detection (Linux/macOS/Windows/FreeBSD/Android)
2. ✅ Architecture detection (x64/ARM64)
3. ✅ Library loading from `core/ffi/libs/{platform}/`
4. ✅ Automatic memory cleanup (weakref/Cleaner/deinit)
5. ✅ Error handling with descriptive messages

**Platform Matrix** (FS-Enabled):

| Platform | Arch | Library | Mandatory Support |
|----------|------|---------|-------------------|
| Linux | x64 | libgbln.so | ✅ Mandatory |
| Linux | ARM64 | libgbln.so | ✅ Mandatory |
| macOS | ARM64 | libgbln.dylib | ✅ Mandatory |
| macOS | x64 | libgbln.dylib | ✅ Mandatory |
| Windows | x64 | gbln.dll | ✅ Mandatory |
| FreeBSD | x64 | libgbln.so | ✅ Mandatory |
| FreeBSD | ARM64 | libgbln.so | ✅ Mandatory |
| Android | ARM64 | libgbln.so | ✅ Mandatory |

---

## Implementation Checklist

Use this checklist for every new language binding:

### Phase 1: Setup
- [ ] Determine category (FS-enabled vs FS-disabled)
- [ ] Create directory structure matching reference
- [ ] Set up build system (matching language ecosystem)
- [ ] Add LICENSE (MIT)
- [ ] Create README.md with usage examples

### Phase 2: Core Implementation (FS-Enabled)
- [ ] FFI wrapper with platform detection
- [ ] Value conversion (language types ↔ GBLN)
- [ ] Parser module with `parse()` and `parseFile()`
- [ ] Serialiser module with `toString()` and `toStringPretty()`
- [ ] **IO module with `readFile()` and `writeFile()`**
- [ ] Config module with configuration options
- [ ] Error module with custom exception types

### Phase 2: Core Implementation (FS-Disabled)
- [ ] Parser module with `parse()` (string only)
- [ ] Serialiser module with `toString()` and `toStringPretty()`
- [ ] Value conversion (language types ↔ GBLN)
- [ ] Error module with custom error types

### Phase 3: Testing
- [ ] `test_parse` - Primitives, objects, arrays
- [ ] `test_serialise` - String output, formatting
- [ ] `test_io` - File operations (FS-enabled only)
- [ ] `test_objects` - Object/array handling
- [ ] `test_ffi` - Library loading
- [ ] Achieve >70% code coverage
- [ ] Add test fixtures (example .gbln files)

### Phase 4: Documentation
- [ ] API documentation in README
- [ ] Usage examples for all features
- [ ] Platform support matrix
- [ ] Installation instructions
- [ ] Contribution guidelines

### Phase 5: Quality Gates
- [ ] All files <400 lines (except FFI <500)
- [ ] All tests passing
- [ ] Code coverage >70%
- [ ] No TODO/FIXME in production code
- [ ] BBC English in all comments
- [ ] Type hints/annotations (if language supports)

---

## Common Patterns

### Platform Detection (FFI Layer)

**Python Pattern** (Reference):
```python
def _find_library() -> Optional[Path]:
    system = platform.system()
    machine = platform.machine()
    
    if system == "Linux":
        if machine in ("x86_64", "AMD64"):
            lib_dir = "linux-x64"
            lib_name = "libgbln.so"
        elif machine in ("aarch64", "arm64"):
            lib_dir = "linux-arm64"
            lib_name = "libgbln.so"
    # ... etc
```

**Kotlin Pattern** (Adapt from Python):
```kotlin
private fun findLibrary(): Path? {
    val osName = System.getProperty("os.name").lowercase()
    val osArch = System.getProperty("os.arch").lowercase()
    
    val (libDir, libName) = when {
        osName.contains("linux") && osArch in listOf("x86_64", "amd64") -> 
            "linux-x64" to "libgbln.so"
        // ... etc
    }
}
```

### Memory Management

**Python** (weakref):
```python
class ManagedGblnValue:
    def __init__(self, ptr):
        self._ptr = ptr
        self._finaliser = weakref.finalize(self, _lib.gbln_value_free, ptr)
```

**Kotlin** (Cleaner):
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

**Swift** (deinit):
```swift
class ManagedGblnValue {
    private let ptr: UnsafeMutablePointer<GblnValue>
    
    deinit {
        gbln_value_free(ptr)
    }
}
```

---

## Gaps Identified

### Kotlin Bindings - Action Items

**Priority: HIGH - Missing mandatory modules for FS-enabled binding**

1. **Add io.kt** (100-200 lines)
   ```kotlin
   fun readFile(path: String): Any?
   fun readFile(path: Path): Any?
   fun writeFile(path: String, value: Any?)
   fun writeFile(path: Path, value: Any?)
   ```

2. **Add Config.kt** (50-100 lines)
   ```kotlin
   data class GblnConfig(
       val miniMode: Boolean = true,
       val compress: Boolean = false,
       val compressionLevel: Int = 6,
       val indent: Int = 2
   )
   ```

3. **Add Error.kt** (50-100 lines)
   ```kotlin
   sealed class GblnError(message: String) : Exception(message)
   class ParseError(message: String) : GblnError(message)
   class SerialiseError(message: String) : GblnError(message)
   class IoError(message: String) : GblnError(message)
   ```

4. **Add comprehensive tests**
   - `IoTest.kt` - File read/write tests
   - `SerialiserTest.kt` - Serialisation tests
   - `ObjectTest.kt` - Object/array tests
   - `ValueConversionTest.kt` - Type conversion tests

5. **Add test fixtures**
   - `tests/fixtures/simple.gbln`
   - `tests/fixtures/nested.gbln`
   - `tests/fixtures/array.gbln`

### Swift Bindings - Verification Needed

**Priority: MEDIUM - Appears complete, needs verification**

1. Run full test suite
2. Verify all 7 modules present and <400 lines
3. Check test coverage percentage
4. Verify README is complete

### JavaScript Bindings - Documentation

**Priority: LOW - Complete for FS-disabled, needs clarity**

1. Update README to explicitly state "FS-disabled" category
2. Document why io module is not present (browser security)
3. Note: Node.js users wanting file I/O should use a different approach (not WASM)

---

## Recommendations

### For Future Bindings

1. **Always start from category decision**:
   - Ask: "Does this language/platform have filesystem access?"
   - FS-enabled (native) → Full feature set
   - FS-disabled (browser/WASM) → Limited feature set

2. **Use Python as reference for FS-enabled**:
   - Copy file structure exactly
   - Adapt code to language idioms
   - Maintain module names (parse, serialise, io, etc.)

3. **Use JavaScript as reference for FS-disabled**:
   - Omit io module
   - Focus on parse + serialise
   - Document limitations clearly

4. **Establish quality gates early**:
   - Set up file size checks in CI
   - Require test coverage >70%
   - Block commits with files >400 lines

---

## Version History

**v1.0** (2025-11-29)
- Initial quality checkup
- Identified FS-enabled vs FS-disabled categories
- Documented Kotlin gaps
- Established mandatory standards

---

**Maintained by**: GBLN Project  
**Contact**: ask@vvoss.dev
