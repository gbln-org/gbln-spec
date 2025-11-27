# GBLN Language Bindings - Build & Test Guide

**Version:** 1.0  
**Last Updated:** 2025-11-27  
**Status:** Active

---

## ⚠️ MANDATORY POLICY

**ALL language bindings MUST follow the Docker-based build and test strategy.**

This document defines the **ONLY acceptable** way to build and test language bindings across all platforms.

---

## Overview

GBLN provides bindings for 12+ languages, all using the C FFI layer as foundation:

```
Rust Core (#004) → C FFI (#005 + #005B) → Language Bindings (#100-111) → User API
```

**All bindings share:**
- Same C FFI library (`libgbln.so`, `libgbln.dylib`, `gbln.dll`)
- Same platform support (10 platforms)
- Same quality requirements (Docker-tested on all platforms)
- Same documentation patterns

---

## Reference Implementation

**Ticket #100 (Python bindings)** is the reference implementation for ALL other bindings.

**Before starting any binding ticket (#101-111):**

1. ✅ Read Ticket #100 completely
2. ✅ Study `bindings/python/` implementation
3. ✅ Review `docs/BINDING_REFERENCE.md` (created during #100)
4. ✅ Review `docs/PLATFORM_TESTING.md` (created during #100)
5. ✅ Copy `bindings/TEMPLATE/` as starting point

---

## Platform Matrix (MANDATORY)

**Every binding MUST be tested on ALL platforms before ticket completion.**

| Platform | Architectures | Docker Image Base | Status |
|----------|---------------|-------------------|--------|
| FreeBSD | x86_64, ARM64 | `freebsd:13.2` | Required |
| Linux | x86_64, ARM64 | Language-specific (e.g., `python:3.11-slim`) | Required |
| macOS | x86_64, ARM64 | N/A (pending Docker support) | Native exception |
| Windows | x86_64 | `mcr.microsoft.com/windows` | Required |
| Android | ARM64, x86_64 | Termux-based | Required |
| iOS | ARM64 | N/A (pending Docker support) | Future |

**Required Platforms:** 7-9 (depending on Docker image availability)  
**Blocking:** Ticket cannot be completed if any required platform fails

---

## Build Infrastructure (Per Binding)

**Location:** `bindings/{language}/tests/docker/`

### Required Files

```
bindings/{language}/
├── tests/
│   └── docker/
│       ├── test-all-platforms.sh       # Master test script (MANDATORY)
│       ├── Dockerfile.freebsd-x64      # FreeBSD x86_64
│       ├── Dockerfile.freebsd-arm64    # FreeBSD ARM64
│       ├── Dockerfile.linux-x64        # Linux x86_64
│       ├── Dockerfile.linux-arm64      # Linux ARM64
│       ├── Dockerfile.windows-x64      # Windows x86_64
│       ├── Dockerfile.android-arm64    # Android ARM64
│       ├── Dockerfile.android-x64      # Android x86_64
│       └── README.md                   # Docker test documentation
```

### Master Test Script Template

**File:** `bindings/{language}/tests/docker/test-all-platforms.sh`

```bash
#!/usr/bin/env bash
# Test {LANGUAGE} bindings on ALL platforms via Docker
# This is the MANDATORY test method

set -euo pipefail

cd "$(dirname "$0")"

PLATFORMS=(
    "freebsd-x64"
    "freebsd-arm64"
    "linux-x64"
    "linux-arm64"
    "windows-x64"
    "android-arm64"
    "android-x64"
)

echo "GBLN {LANGUAGE} Bindings - Platform Test Suite"
echo "=============================================="
echo "⚠️  MANDATORY: All platforms must pass"
echo ""

FAILED=()

for platform in "${PLATFORMS[@]}"; do
    echo ""
    echo "Testing: $platform"
    echo "-------------------"
    
    if docker build -f "Dockerfile.$platform" -t "gbln-{language}:$platform" ../../.. && \
       docker run --rm "gbln-{language}:$platform"; then
        echo "✓ $platform: PASSED"
    else
        echo "✗ $platform: FAILED"
        FAILED+=("$platform")
    fi
done

echo ""
echo "========================================="
if [ ${#FAILED[@]} -eq 0 ]; then
    echo "✓ ALL PLATFORMS PASSED (${#PLATFORMS[@]}/${#PLATFORMS[@]})"
    echo "Ticket #{TICKET_NUM} platform testing: COMPLETE"
    exit 0
else
    echo "✗ FAILURES: ${#FAILED[@]}/${#PLATFORMS[@]}"
    for platform in "${FAILED[@]}"; do
        echo "  - $platform"
    done
    echo ""
    echo "Ticket #{TICKET_NUM} platform testing: INCOMPLETE"
    exit 1
fi
```

---

## Dockerfile Templates

### Template: Linux x86_64

```dockerfile
FROM {language_base_image}

# Install libgbln.so for x86_64-unknown-linux-gnu
COPY core/ffi/target/x86_64-unknown-linux-gnu/release/libgbln.so /usr/lib/

# Copy binding package
WORKDIR /app
COPY bindings/{language}/ /app/

# Install dependencies and package
RUN {language_install_command}

# Run tests
CMD ["{language_test_command}"]
```

### Template: FreeBSD x86_64

```dockerfile
FROM freebsd:13.2

# Install build dependencies
RUN pkg update && pkg install -y {language_runtime}

# Install libgbln.so for x86_64-unknown-freebsd
COPY core/ffi/target/x86_64-unknown-freebsd/release/libgbln.so /usr/lib/

# Copy binding package
WORKDIR /app
COPY bindings/{language}/ /app/

# Install package
RUN {language_install_command}

# Run tests
CMD ["{language_test_command}"]
```

### Template: Windows x86_64

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2022

# Install language runtime
RUN {windows_install_runtime}

# Copy gbln.dll for x86_64-pc-windows-gnu
COPY core/ffi/target/x86_64-pc-windows-gnu/release/gbln.dll C:/Windows/System32/

# Copy binding package
WORKDIR C:/app
COPY bindings/{language}/ C:/app/

# Install package
RUN {language_install_command}

# Run tests
CMD ["{language_test_command}"]
```

---

## Language-Specific Adaptations

### Python (#100)

**Base Image:** `python:3.11-slim`  
**Install:** `pip install -e .[dev]`  
**Test:** `pytest -v --tb=short`  
**Library Loading:** `ctypes.CDLL()`

### JavaScript (#101)

**Base Image:** `node:20-slim`  
**Install:** `npm install`  
**Test:** `npm test`  
**Library Loading:** WASM module compiled from C FFI

### Swift (#102)

**Base Image:** `swift:5.9`  
**Install:** `swift build`  
**Test:** `swift test`  
**Library Loading:** `dlopen()` + Swift/C interop

### Kotlin (#103)

**Base Image:** `eclipse-temurin:17-jdk`  
**Install:** `./gradlew build`  
**Test:** `./gradlew test`  
**Library Loading:** JNA

### Go (#104)

**Base Image:** `golang:1.21`  
**Install:** `go build`  
**Test:** `go test ./...`  
**Library Loading:** CGO

### Java (#105)

**Base Image:** `eclipse-temurin:17-jdk`  
**Install:** `mvn install`  
**Test:** `mvn test`  
**Library Loading:** JNA

### C# (#106)

**Base Image:** `mcr.microsoft.com/dotnet/sdk:8.0`  
**Install:** `dotnet build`  
**Test:** `dotnet test`  
**Library Loading:** P/Invoke

### C++ (#107)

**Base Image:** `gcc:13`  
**Install:** `cmake --build build`  
**Test:** `ctest --test-dir build`  
**Library Loading:** Native `dlopen()`

### Ruby (#108)

**Base Image:** `ruby:3.2`  
**Install:** `bundle install`  
**Test:** `bundle exec rspec`  
**Library Loading:** FFI gem

### PHP (#109)

**Base Image:** `php:8.2-cli`  
**Install:** `composer install`  
**Test:** `vendor/bin/phpunit`  
**Library Loading:** FFI extension

### Perl (#110)

**Base Image:** `perl:5.38`  
**Install:** `cpanm --installdeps .`  
**Test:** `prove -l t/`  
**Library Loading:** `FFI::Platypus`

### Tcl (#111)

**Base Image:** `tcl:8.6`  
**Install:** N/A (script language)  
**Test:** `tclsh tests/all.tcl`  
**Library Loading:** `load` command

---

## libgbln Distribution Strategy

**ALL bindings need libgbln binaries for all platforms.**

### Option 1: Bundled in Package (Recommended)

Include precompiled `libgbln.*` for all platforms in the language package:

```
bindings/{language}/
├── lib/
│   ├── freebsd/
│   │   ├── x86_64/libgbln.so
│   │   └── aarch64/libgbln.so
│   ├── linux/
│   │   ├── x86_64/libgbln.so
│   │   └── aarch64/libgbln.so
│   ├── macos/
│   │   ├── x86_64/libgbln.dylib
│   │   └── aarch64/libgbln.dylib
│   ├── windows/
│   │   └── x86_64/gbln.dll
│   └── android/
│       ├── aarch64/libgbln.so
│       └── x86_64/libgbln.so
```

**Pros:** Self-contained, works offline  
**Cons:** Larger package size

### Option 2: Download on Install

Download platform-specific binary during package installation:

**Pros:** Smaller package size  
**Cons:** Requires internet, install complexity

### Option 3: System Library

Expect user to install `libgbln` separately:

**Pros:** Smallest package  
**Cons:** User burden, version mismatch issues

**Recommendation:** Use **Option 1** for user-facing bindings (Python, JavaScript, etc.)

---

## Testing Requirements

### Minimum Test Coverage

**Every binding MUST have:**

1. **Unit Tests (40%)** - Individual function tests
2. **Integration Tests (40%)** - Parse → Serialize → Parse round-trips
3. **I/O Tests (20%)** - write_io → read_io round-trips
4. **Platform Tests** - Verify library loading on each platform

**Minimum:** 100+ test cases

### Test Categories

```
tests/
├── unit/
│   ├── test_ffi_loading.{ext}       # Library loading
│   ├── test_value_conversion.{ext}  # Type conversion
│   ├── test_memory_management.{ext} # No leaks
│   └── test_error_handling.{ext}    # All error cases
├── integration/
│   ├── test_parse.{ext}             # Parse GBLN strings
│   ├── test_serialize.{ext}         # Serialize to GBLN
│   └── test_roundtrip.{ext}         # Full round-trip
├── io/
│   ├── test_write_io.{ext}          # I/O format writing
│   └── test_read_io.{ext}           # I/O format reading
└── platform/
    ├── test_library_exists.{ext}    # Platform-specific
    └── test_utf8_handling.{ext}     # Unicode support
```

### Acceptance Criteria

**Before marking ticket complete:**

- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] All I/O tests pass
- [ ] All platform tests pass
- [ ] **MANDATORY:** `test-all-platforms.sh` passes for ALL required platforms
- [ ] No memory leaks (verified on at least Linux x64)
- [ ] UTF-8 handling correct on all platforms
- [ ] Error messages correct on all platforms

---

## CI/CD Integration

### GitHub Actions Workflow Template

```yaml
name: Platform Tests

on: [push, pull_request]

jobs:
  test-docker-platforms:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        platform:
          - freebsd-x64
          - freebsd-arm64
          - linux-x64
          - linux-arm64
          - windows-x64
          - android-arm64
          - android-x64
    
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive
      
      - name: Build Docker image
        run: |
          cd bindings/{language}/tests/docker
          docker build -f Dockerfile.${{ matrix.platform }} \
            -t gbln-{language}:${{ matrix.platform }} \
            ../../..
      
      - name: Run tests
        run: |
          docker run --rm gbln-{language}:${{ matrix.platform }}
  
  test-native:
    strategy:
      matrix:
        os: [macos-latest]
        arch: [arm64]
    
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Install {language}
        run: {install_command}
      
      - name: Build libgbln
        run: |
          cd core/ffi
          cargo build --release
      
      - name: Install package
        run: |
          cd bindings/{language}
          {install_command}
      
      - name: Run tests
        run: {test_command}
```

---

## Documentation Requirements

### README.md Template

Every binding MUST include:

```markdown
# GBLN - {Language} Bindings

[![Platform](https://img.shields.io/badge/FreeBSD-x64%20%7C%20ARM64-red)](https://www.freebsd.org/)
[![Platform](https://img.shields.io/badge/Linux-x64%20%7C%20ARM64-blue)](https://www.kernel.org/)
[![Platform](https://img.shields.io/badge/macOS-x64%20%7C%20ARM64-lightgrey)](https://www.apple.com/macos/)
[![Platform](https://img.shields.io/badge/Windows-x64-blue)](https://www.microsoft.com/windows/)
[![Platform](https://img.shields.io/badge/Android-ARM64%20%7C%20x64-green)](https://www.android.com/)
[![Docker Tested](https://img.shields.io/badge/Docker-Tested%20on%209%20platforms-blue)](#platform-testing)

## Installation

{language_specific_install}

## Quick Start

{language_specific_examples}

## Platform Support

Tested on all platforms via Docker. See [Platform Testing](#platform-testing).

## Platform Testing

All tests pass on:
- ✅ FreeBSD x86_64, ARM64
- ✅ Linux x86_64, ARM64
- ✅ Windows x86_64
- ✅ Android ARM64, x86_64
- ⚠️ macOS x86_64, ARM64 (native tested)

See `tests/docker/README.md` for running platform tests locally.
```

---

## Common Patterns

### FFI Library Loading

**Pattern from #100 (Python):**

```python
def _find_library():
    # 1. Environment variable GBLN_LIBRARY_PATH
    # 2. Alongside package (bundled)
    # 3. System library paths
    # 4. Error with helpful message
```

**Adapt to each language's idioms.**

### Memory Management

**C FFI returns pointers that MUST be freed:**

- `GblnValue*` → `gbln_value_free()`
- `GblnString*` → `gbln_string_free()`
- `char**` (keys) → `gbln_keys_free()`

**Language-specific solutions:**
- Python: `weakref.finalize()`
- JavaScript: `FinalizationRegistry`
- Swift: `deinit`
- Go: `runtime.SetFinalizer()`
- Java: `try-with-resources`
- C++: RAII with destructors

### Type Conversion

**GBLN ↔ Native type mapping:**

| GBLN Type | Python | JavaScript | Java | Go | C++ |
|-----------|--------|------------|------|-----|-----|
| I8-I64 | `int` | `number` | `long` | `int64` | `int64_t` |
| U8-U64 | `int` | `number` | `long` | `uint64` | `uint64_t` |
| F32, F64 | `float` | `number` | `double` | `float64` | `double` |
| Str | `str` | `string` | `String` | `string` | `std::string` |
| Bool | `bool` | `boolean` | `boolean` | `bool` | `bool` |
| Null | `None` | `null` | `null` | `nil` | `nullptr` |
| Object | `dict` | `object` | `Map` | `map` | `std::map` |
| Array | `list` | `array` | `List` | `[]T` | `std::vector` |

---

## Quality Checklist

**Before submitting binding ticket for review:**

- [ ] All code follows language idioms
- [ ] All comments in BBC English
- [ ] No files >400 lines
- [ ] No generic names (`utils`, `helpers`, `common`)
- [ ] Type annotations/hints where supported
- [ ] Memory management verified (no leaks)
- [ ] All tests pass locally
- [ ] **Docker test infrastructure created**
- [ ] **`test-all-platforms.sh` passes**
- [ ] README with platform badges
- [ ] Platform testing documentation
- [ ] Installation instructions for all platforms
- [ ] Usage examples
- [ ] API documentation

---

## Ticket Dependencies

**Before starting any binding ticket:**

1. ✅ #004 (Rust Core) - COMPLETED
2. ✅ #005 (C FFI) - COMPLETED
3. ✅ #005B (C FFI Extensions) - COMPLETED
4. ✅ #100 (Python - Reference) - IN PROGRESS

**Recommended order:**

1. #100 Python (Reference implementation)
2. #101 JavaScript (High priority, wide usage)
3. #102 Swift (iOS/macOS native)
4. #103 Kotlin (Android/JVM native)
5. #104 Go (Backend/CLI usage)
6. #105-111 (Medium/Low priority)

---

## Support

**Questions?**
- Study Ticket #100 implementation
- Read `docs/BINDING_REFERENCE.md`
- Read `docs/PLATFORM_TESTING.md`
- Check `bindings/TEMPLATE/`

**Issues?**
- File in respective language binding repo
- Reference parent ticket number
- Include platform test results

---

**Last Updated:** 2025-11-27  
**Policy Version:** 1.0  
**Reference Ticket:** #100 (Python bindings)
