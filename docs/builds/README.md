# GBLN Build Infrastructure

**Version:** 2.0  
**Last Updated:** 2025-11-28  
**Status:** Production - VERIFIED

This directory contains documentation for building GBLN on all supported platforms.

---

## ⚠️ MANDATORY BUILD POLICY

**For C FFI Libraries: There is ONLY ONE valid way to build - the `cross` tool.**

**For Language Bindings: Use pre-built libraries from `core/ffi/libs/`.**

This is not a recommendation. This is the **ONLY method that works**.

---

## Documentation

**[BUILD_SYSTEM.md](BUILD_SYSTEM.md)** - Complete C FFI build system (v4.0 - cross tool)  
**[BINDING_BUILDS.md](BINDING_BUILDS.md)** - Language binding build guide (v2.0)

---

## Overview

GBLN supports **8 platforms** with **pre-built C FFI libraries**:

| Platform | Architecture | Library | Size | Build Method | Status |
|----------|--------------|---------|------|--------------|--------|
| Linux | x86_64 | libgbln.so | 573K | cross | ✅ |
| Linux | ARM64 | libgbln.so | 583K | cross | ✅ |
| FreeBSD | x86_64 | libgbln.so | 548K | cross | ✅ |
| FreeBSD | ARM64 | libgbln.so | 639K | cross +nightly | ✅ |
| Windows | x86_64 | gbln.dll | 1.5M | cross | ✅ |
| Android | ARM64 | libgbln.so | 627K | cross | ✅ |
| Android | x86_64 | libgbln.so | 617K | cross | ✅ |
| macOS | ARM64 | libgbln.dylib | 553K | cargo (native) | ✅ |

**All libraries verified:** 2025-11-28  
**Location:** `core/ffi/libs/` (committed to Git)

---

## Quick Start

### For C FFI Development

**Build all platforms:**
```bash
# Prerequisites: Docker + rustup + cross
cd core/ffi
./build-all.sh
```

**Build single platform:**
```bash
cd core/ffi
cross build --release --target x86_64-unknown-linux-gnu
```

See **[BUILD_SYSTEM.md](BUILD_SYSTEM.md)** for details.

### For Language Binding Development

**DO NOT build C FFI!** Use pre-built libraries:

```python
# Python example
import ctypes
from pathlib import Path

# Load from core/ffi/libs/{platform}/
lib_path = Path(__file__).parent / "../../core/ffi/libs/linux-x64/libgbln.so"
lib = ctypes.CDLL(str(lib_path))
```

See **[BINDING_BUILDS.md](BINDING_BUILDS.md)** for details.

---

## Build Architecture

```
┌─────────────────────────────────────────────────┐
│ C FFI Build (ONCE, by core developers)         │
│ - Uses cross tool                               │
│ - Builds all 8 platforms                        │
│ - Commits to Git: core/ffi/libs/                │
└─────────────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────┐
│ Pre-Built Libraries (in Git)                    │
│ core/ffi/libs/{platform}/lib*.{so|dll|dylib}    │
└─────────────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────┐
│ Language Bindings (load pre-built libs)         │
│ - Python, JavaScript, Go, Swift, etc.           │
│ - Platform detection + library loading          │
│ - NO compilation needed                         │
└─────────────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────┐
│ User Installation                               │
│ - pip install gbln / npm install gbln           │
│ - Gets working binaries                         │
│ - NO build tools needed                         │
└─────────────────────────────────────────────────┘
```

**Key Principle:** Build once, distribute everywhere.

---

## Build Methods

### C FFI Libraries

**Method:** `cross` tool (Rust cross-compilation manager)

**Why `cross`:**
- ✅ Proven to work (all 8 platforms verified)
- ✅ Handles exotic targets (FreeBSD, Android, Windows)
- ✅ Supports `-Z build-std` for FreeBSD ARM64
- ✅ Uses Docker internally with correct toolchains
- ✅ Standard tool in Rust ecosystem

**NOT allowed:**
- ❌ Pure Docker (lacks Rust target support)
- ❌ Native builds for non-host platforms
- ❌ Manual cross-compilation

**Build frequency:** Rarely (only when C FFI changes)

### Language Bindings

**Method:** Use pre-built libraries from `core/ffi/libs/`

**Why pre-built:**
- ✅ Users don't need Rust/Docker/build tools
- ✅ Instant installation (`pip install gbln` just works)
- ✅ Consistent binaries across all users
- ✅ Standard practice for binary distributions

**NOT allowed:**
- ❌ Building C FFI during binding development
- ❌ Expecting users to compile C FFI
- ❌ Using libraries from `target/` directory

**Build frequency:** Never (use pre-built libraries)

---

## Repository Structure

```
GBLN/
├── core/
│   ├── rust/                    # Rust parser (dependency for FFI)
│   └── ffi/                     # C FFI layer
│       ├── src/lib.rs           # FFI implementation
│       ├── build-all.sh         # Build script (uses cross)
│       ├── libs/                # Pre-built libraries (Git)
│       │   ├── linux-x64/
│       │   ├── linux-arm64/
│       │   ├── freebsd-x64/
│       │   ├── freebsd-arm64/
│       │   ├── windows-x64/
│       │   ├── android-arm64/
│       │   ├── android-x64/
│       │   └── macos-arm64/
│       └── target/              # Build artifacts (ignored by Git)
│
├── bindings/
│   ├── python/                  # References core/ffi/libs/
│   ├── javascript/              # References core/ffi/libs/
│   ├── go/                      # References core/ffi/libs/
│   └── ...                      # All use core/ffi/libs/
│
└── docs/
    └── builds/                  # This directory
        ├── README.md            # This file
        ├── BUILD_SYSTEM.md      # C FFI build documentation
        └── BINDING_BUILDS.md    # Language binding documentation
```

---

## Common Tasks

### Rebuild C FFI (Core Developers Only)

```bash
# Prerequisites
brew install --cask docker
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
cargo install cross --git https://github.com/cross-rs/cross
rustup install nightly

# Build all platforms
cd core/ffi
./build-all.sh

# Organize libraries
mkdir -p libs/{linux-x64,linux-arm64,freebsd-x64,freebsd-arm64,windows-x64,android-arm64,android-x64,macos-arm64}
cp target/*/release/lib*.{so,dll,dylib} libs/*/

# Commit to Git
git add libs/
git commit -m "Update C FFI libraries"
```

### Develop Language Binding

```bash
# Just use pre-built libraries!
cd bindings/python

# Libraries are already in core/ffi/libs/
# Your code loads them based on platform detection

# Test
pytest -v

# Package (includes all libraries)
python setup.py sdist bdist_wheel
```

### Test on Different Platform

```bash
# Libraries auto-detect platform
cd bindings/python
pytest -v

# On Linux x64: loads core/ffi/libs/linux-x64/libgbln.so
# On macOS ARM64: loads core/ffi/libs/macos-arm64/libgbln.dylib
# On Windows x64: loads core/ffi/libs/windows-x64/gbln.dll
```

---

## Troubleshooting

### "Library not found" Error

**Check:**
1. Is `core/ffi/libs/{platform}/` present?
2. Is platform detection correct?
3. Is library path calculation correct?

**Solution:**
```bash
# Verify libraries exist
ls -lh core/ffi/libs/*/

# Check platform detection
python -c "import platform; print(platform.system(), platform.machine())"
```

### Building C FFI Fails

**Common issues:**
1. Docker not running → `open -a Docker`
2. `cross` not installed → `cargo install cross --git https://github.com/cross-rs/cross`
3. Parallel builds → Kill all processes, rebuild sequentially

**Solution:** See [BUILD_SYSTEM.md](BUILD_SYSTEM.md) Troubleshooting section

---

## CI/CD Integration

**GitHub Actions** (planned):

```yaml
name: Test Bindings

on: [push, pull_request]

jobs:
  test-python:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Install package
        run: |
          cd bindings/python
          pip install -e .[dev]
      
      - name: Run tests
        run: |
          cd bindings/python
          pytest -v
```

**Note:** Libraries are in Git, so CI just installs and tests. No build step!

---

## Summary

**Build System Status:** ✅ Production Ready

- **C FFI:** Built with `cross`, all 8 platforms verified
- **Libraries:** Committed to Git in `core/ffi/libs/`
- **Bindings:** Use pre-built libraries, no compilation
- **Users:** Install packages that just work

**Key Insights:**
1. `cross` is mandatory for C FFI (pure Docker failed)
2. Pre-built libraries must be in Git (for distribution)
3. Language bindings load platform-specific library
4. Users get working binaries, zero build time

**Lessons Learned:** Hours wasted trying Docker-only approach. `cross` is the proven solution.

---

**Version:** 2.0  
**Last Updated:** 2025-11-28  
**Build Policy:** Use `cross` for C FFI, pre-built libs for bindings  
**Maintained by:** GBLN Project  
**Contact:** ask@vvoss.dev
