# GBLN Build System Documentation

**Version:** 4.0  
**Last Updated:** 2025-11-28  
**Status:** Production - VERIFIED & TESTED

---

## ⚠️ MANDATORY BUILD POLICY ⚠️

**CRITICAL: There is ONLY ONE VALID way to build C FFI libraries for all platforms.**

### The Only Valid Build Method

**ALL cross-platform C FFI builds MUST use the `cross` tool.**

This is **NOT optional**. This is **NOT a recommendation**. This is the **ONLY method that works**.

### Why `cross` Is Mandatory

1. **Proven**: Successfully built all 8 platforms (verified 2025-11-28)
2. **Rust Ecosystem Standard**: Official Rust cross-compilation tool
3. **Docker-Based**: Uses Docker internally with correct toolchains
4. **Target Support**: Handles FreeBSD, Android, Windows cross-compilation
5. **Build-std Support**: Works with `-Z build-std` for exotic targets (FreeBSD ARM64)

### What Is NOT Allowed

❌ **Pure Docker** (lacks Rust target support, requires manual toolchain setup)  
❌ **Native cargo build** for non-host platforms (doesn't work)  
❌ **Manual cross-compilation** (too complex, error-prone)  
❌ **zig cc** or other experimental tools (unproven for our use case)

### Critical Lesson Learned

**We tried pure Docker.** It failed. Hours wasted. `cross` is the proven solution.

**Reference:** See `core/ffi/BUILD_RESULTS.md` for successful build verification.

---

## Overview

This document describes the GBLN C FFI build system for all 8 supported platforms.

**Supported Platforms:** 8  
**Build Method:** `cross` tool (mandatory)  
**Pre-built Libraries:** Committed to Git in `core/ffi/libs/`  
**Distribution:** Ready-to-use binaries in language binding packages

---

## Build Architecture

```
Host: macOS/Linux with Docker + rustup + cross
   ↓
┌──────────────────────────────────────────────────────┐
│  cross Tool (Rust Cross-Compilation Manager)         │
│  - Manages Docker containers internally              │
│  - Provides correct Rust toolchains per target       │
│  - Handles build-std for exotic targets              │
└──────────────────────────────────────────────────────┘
   ↓
┌──────────────────────────────────────────────────────┐
│  Platform-Specific Docker Containers (via cross)     │
│  - FreeBSD x86_64, ARM64                             │
│  - Linux x86_64, ARM64                               │
│  - Windows x86_64                                    │
│  - Android ARM64, x86_64                             │
│  - macOS ARM64 (native)                              │
└──────────────────────────────────────────────────────┘
   ↓
┌──────────────────────────────────────────────────────┐
│  Pre-Built Libraries (core/ffi/libs/)                │
│  - Committed to Git                                  │
│  - Used by ALL language bindings                     │
│  - Distributed in packages (pip, npm, gem, etc.)     │
└──────────────────────────────────────────────────────┘
```

**Key Principle:** Use `cross`, build sequentially (lock files!), commit to Git.

---

## Prerequisites

### Required Tools

1. **Docker Desktop**:
   ```bash
   # macOS
   brew install --cask docker
   
   # Verify
   docker --version
   docker ps
   ```

2. **rustup** (Rust toolchain manager):
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
   
   # Add to PATH
   source "$HOME/.cargo/env"
   
   # Verify
   rustup --version
   ```

3. **cross** (cross-compilation tool):
   ```bash
   cargo install cross --git https://github.com/cross-rs/cross
   
   # Verify
   cross --version
   ```

4. **Nightly toolchain** (for FreeBSD ARM64):
   ```bash
   rustup install nightly
   ```

---

## Platform Matrix

**All 8 platforms are MANDATORY for all language bindings.**

| Platform | Target Triple | Method | Library | Size | Status |
|----------|--------------|--------|---------|------|--------|
| Linux x86_64 | `x86_64-unknown-linux-gnu` | cross | libgbln.so | 573K | ✅ Mandatory |
| Linux ARM64 | `aarch64-unknown-linux-gnu` | cross | libgbln.so | 583K | ✅ Mandatory |
| FreeBSD x86_64 | `x86_64-unknown-freebsd` | cross | libgbln.so | 548K | ✅ Mandatory |
| FreeBSD ARM64 | `aarch64-unknown-freebsd` | cross +nightly | libgbln.so | 639K | ✅ Mandatory |
| Windows x86_64 | `x86_64-pc-windows-gnu` | cross | gbln.dll | 1.5M | ✅ Mandatory |
| Android ARM64 | `aarch64-linux-android` | cross | libgbln.so | 627K | ✅ Mandatory |
| Android x86_64 | `x86_64-linux-android` | cross | libgbln.so | 617K | ✅ Mandatory |
| macOS ARM64 | `aarch64-apple-darwin` | native | libgbln.dylib | 553K | ✅ Mandatory |

**All builds verified:** 2025-11-28  
**Quality Standards:** See `BINDING_BUILDS_QS_NOTES.md` for binding implementation requirements

---

## Building C FFI Libraries

### ⚠️ CRITICAL: USE build-all.sh SCRIPT ONLY ⚠️

**THIS IS THE ONLY VALID WAY TO BUILD C FFI LIBRARIES.**

```bash
cd core/ffi
./build-all.sh
```

**DO NOT:**
- ❌ Run builds manually in parallel
- ❌ Use background jobs (`&`)
- ❌ Run multiple `cross build` commands simultaneously
- ❌ Modify this script without testing all 8 platforms

**WHY:**
- Cargo lock files cause parallel builds to hang indefinitely
- Builds will block waiting for locks with 0% CPU usage
- This has been proven multiple times - sequential is the ONLY way

### What build-all.sh Does

The script builds all 8 platforms **sequentially** in this exact order:

```bash
# 1. Linux x86_64
cross build --release --target x86_64-unknown-linux-gnu

# 2. Linux ARM64
cross build --release --target aarch64-unknown-linux-gnu

# 3. FreeBSD x86_64
cross build --release --target x86_64-unknown-freebsd

# 4. FreeBSD ARM64 (requires nightly + build-std)
cross +nightly build --release --target aarch64-unknown-freebsd -Z build-std

# 5. Windows x86_64
cross build --release --target x86_64-pc-windows-gnu

# 6. Android ARM64
cross build --release --target aarch64-linux-android

# 7. Android x86_64
cross build --release --target x86_64-linux-android

# 8. macOS ARM64 (native)
cargo build --release --target aarch64-apple-darwin
```

Then organizes all libraries into `libs/{platform}/` for distribution.

**Script content** (`core/ffi/build-all.sh`):
```bash
#!/usr/bin/env bash
set -e

PATH="$HOME/.cargo/bin:$PATH"

echo "Building C FFI for all platforms (sequential)..."

# Linux
echo "  → Linux x86_64..."
cross build --release --target x86_64-unknown-linux-gnu

echo "  → Linux ARM64..."
cross build --release --target aarch64-unknown-linux-gnu

# FreeBSD
echo "  → FreeBSD x86_64..."
cross build --release --target x86_64-unknown-freebsd

echo "  → FreeBSD ARM64 (nightly + build-std)..."
cross +nightly build --release --target aarch64-unknown-freebsd -Z build-std

# Windows
echo "  → Windows x86_64..."
cross build --release --target x86_64-pc-windows-gnu

# Android
echo "  → Android ARM64..."
cross build --release --target aarch64-linux-android

echo "  → Android x86_64..."
cross build --release --target x86_64-linux-android

# macOS (native)
echo "  → macOS ARM64 (native)..."
cargo build --release --target aarch64-apple-darwin

echo "✅ All platforms built successfully!"
```

---

## Organizing Libraries

After building, libraries are in `core/ffi/target/{target}/release/`.

**Organize them** for distribution:

```bash
cd core/ffi

# Create directory structure
mkdir -p libs/{linux-x64,linux-arm64,freebsd-x64,freebsd-arm64,windows-x64,android-arm64,android-x64,macos-arm64}

# Copy libraries
cp target/x86_64-unknown-linux-gnu/release/libgbln.so libs/linux-x64/
cp target/aarch64-unknown-linux-gnu/release/libgbln.so libs/linux-arm64/
cp target/x86_64-unknown-freebsd/release/libgbln.so libs/freebsd-x64/
cp target/aarch64-unknown-freebsd/release/libgbln.so libs/freebsd-arm64/
cp target/x86_64-pc-windows-gnu/release/gbln.dll libs/windows-x64/
cp target/aarch64-linux-android/release/libgbln.so libs/android-arm64/
cp target/x86_64-linux-android/release/libgbln.so libs/android-x64/
cp target/aarch64-apple-darwin/release/libgbln.dylib libs/macos-arm64/
```

---

## Committing Libraries to Git

**CRITICAL:** Pre-built libraries MUST be committed to Git.

**Why?**
- Language bindings (Python, JavaScript, etc.) distribute ready-to-use packages
- Users install via `pip install gbln` and get working binaries
- NO compilation required on user machines
- This is standard practice for binary distributions

**How:**

```bash
cd core/ffi

# Verify libs/ is NOT in .gitignore
grep "libs" .gitignore  # Should return nothing

# Add to Git
git add libs/

# Commit
git commit -m "Add pre-built C FFI libraries for all 8 platforms"
```

**Result:** All language bindings reference `core/ffi/libs/` for their platform-specific libraries.

---

## Using Libraries in Language Bindings

### Python Example

```python
import ctypes
import platform
import os

def load_gbln():
    system = platform.system()
    machine = platform.machine()
    
    # Map to library path
    if system == "Linux":
        if machine == "x86_64":
            lib_path = "../../core/ffi/libs/linux-x64/libgbln.so"
        elif machine == "aarch64":
            lib_path = "../../core/ffi/libs/linux-arm64/libgbln.so"
    elif system == "Darwin":
        lib_path = "../../core/ffi/libs/macos-arm64/libgbln.dylib"
    elif system == "Windows":
        lib_path = "../../core/ffi/libs/windows-x64/gbln.dll"
    # ... FreeBSD, Android
    
    return ctypes.CDLL(lib_path)
```

### Package Distribution

**Python wheel:**
```python
# setup.py
from setuptools import setup

setup(
    name="gbln",
    packages=["gbln"],
    package_data={
        "gbln": [
            "libs/linux-x64/*.so",
            "libs/linux-arm64/*.so",
            "libs/windows-x64/*.dll",
            "libs/macos-arm64/*.dylib",
            # ... all platforms
        ]
    },
)
```

**Result:** `pip install gbln` includes all platform libraries, automatically selects correct one.

---

## Build Times

**Approximate times (macOS M1, first build):**

| Platform | Time | Notes |
|----------|------|-------|
| Linux x86_64 | 11s | Fast, cached dependencies |
| Linux ARM64 | 1m 38s | QEMU emulation |
| FreeBSD x86_64 | - | Usually cached |
| FreeBSD ARM64 | 5-10m | Nightly + build-std, compiles std lib |
| Windows x86_64 | 10s | If not hanging (restart if >2min) |
| Android ARM64 | 30s | |
| Android x86_64 | 30s | |
| macOS ARM64 | <5s | Native build |

**Total (sequential, first build): ~15-20 minutes**  
**Subsequent builds:** <5 minutes (cached dependencies)

---

## Troubleshooting

### Build Hangs (File Lock)

**Symptom:** Build shows "Blocking waiting for file lock on build directory", or build runs indefinitely with 0% CPU usage

**Cause:** Multiple `cross build` commands running in parallel trying to access the same Cargo build artifacts

**Root Cause:** Cargo uses lock files (`target/.cargo-lock`) to prevent concurrent access. When multiple cross containers try to build simultaneously, they block each other indefinitely.

**Solution:**
```bash
# 1. Kill all cross processes
pkill -9 -f cross

# 2. Stop ALL Docker containers
docker stop $(docker ps -q)

# 3. Remove stale lock files (if any)
rm -f core/ffi/target/.cargo-lock

# 4. Rebuild sequentially (THE ONLY WAY)
cd core/ffi
./build-all.sh
```

**Prevention:** ALWAYS use `build-all.sh`. NEVER run parallel builds.

**Lesson Learned (2025-11-28):** We tried parallel builds multiple times. They ALWAYS hang. Sequential is the ONLY reliable method.

### FreeBSD ARM64 Toolchain Error

**Symptom:** `toolchain '1.88.0' does not support target 'aarch64-unknown-freebsd'`

**Cause:** FreeBSD ARM64 is a Tier 3 target in Rust - no pre-built standard library available

**Solution:** Use nightly toolchain with `-Z build-std` to compile the standard library from source:
```bash
cross +nightly build --release --target aarch64-unknown-freebsd -Z build-std
```

**Build Time:** 5-10 minutes (first build), as it compiles the entire Rust standard library

**Note:** This is already included in `build-all.sh` for platform 4 (FreeBSD ARM64)

### Docker Permission Denied

```bash
# macOS: Ensure Docker Desktop is running
open -a Docker && sleep 15

# Linux: Add user to docker group
sudo usermod -aG docker $USER
# Then log out and back in
```

### Missing `cross` Command

```bash
# Install via cargo
cargo install cross --git https://github.com/cross-rs/cross

# Add to PATH
export PATH="$HOME/.cargo/bin:$PATH"
```

### macOS Library Load Crashes (Illegal instruction: 4, Killed: 9)

**Symptom:** Python tests crash with `Signal: Killed: 9` or `Illegal instruction: 4` when loading library on macOS

**Cause:** The macOS library has an incorrect `install_name` pointing to an absolute build path instead of `@rpath`

**Diagnosis:**
```bash
otool -L core/ffi/libs/macos-arm64/libgbln.dylib
```

If you see an absolute path like:
```
/Users/.../core/ffi/target/release/deps/libgbln.dylib
```

The library will fail to load.

**Solution:** Fix the install_name using `install_name_tool`:
```bash
install_name_tool -id @rpath/libgbln.dylib core/ffi/libs/macos-arm64/libgbln.dylib
```

**Verification:**
```bash
otool -L core/ffi/libs/macos-arm64/libgbln.dylib
```

Should show:
```
@rpath/libgbln.dylib (compatibility version 0.0.0, current version 0.0.0)
```

**Prevention:** macOS library should always be built natively with `cargo build --release`, not with `cross`. The `build-all.sh` script handles this correctly.

**Lesson Learned (2025-11-28):** Always verify install_name after building macOS libraries. Incorrect install_name causes immediate crashes when ctypes tries to load the library.

---

## CI/CD Integration

**GitHub Actions** (planned):

```yaml
name: Build C FFI

on: [push, pull_request]

jobs:
  build-ffi:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Install Rust
        uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
          
      - name: Install cross
        run: cargo install cross --git https://github.com/cross-rs/cross
        
      - name: Build all platforms
        run: |
          cd core/ffi
          ./build-all.sh
          
      - name: Verify libraries
        run: |
          find core/ffi/libs -type f
```

---

## Summary

**Build System Status:** ✅ Fully Functional & Verified

- **8 platforms** built successfully via `cross`
- **Sequential builds** (avoid lock conflicts)
- **Pre-built libraries** committed to Git (`core/ffi/libs/`)
- **Ready for distribution** in all language bindings
- **User experience:** Install package, it just works (no compilation)

**Key Takeaway:** Use `cross`, not pure Docker. Lessons learned the hard way.

---

## Related Documentation

- **`BINDING_BUILDS.md`** - How to use these libraries in language bindings
- **`BINDING_BUILDS_QS_NOTES.md`** - Quality standards for binding implementations
- **`.claude/tickets/100-gbln-bindings-python-STATUS.md`** - Reference implementation (Python)

---

**Version:** 4.1  
**Last Updated:** 2025-12-01  
**Verified:** All 8 platforms built successfully  
**Maintained by:** GBLN Project  
**Contact:** ask@vvoss.dev
