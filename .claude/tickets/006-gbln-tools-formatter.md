# Ticket #006: CLI Formatter (gbln fmt)

**Repo**: gbln-tools  
**Status**: open  
**Priority**: high  
**Created**: 2025-01-22  
**Updated**: 2025-01-22  

---

## Summary

Create a command-line tool to format GBLN files (compact ↔ pretty-printed).

---

## Requirements

### Core Functionality

1. **Format Modes**
   - Compact: Remove all structural whitespace
   - Pretty: Add indentation for readability
   - Auto-detect current format

2. **Options**
   - `--compact`: Output compact format
   - `--pretty`: Output pretty format (default)
   - `--indent=N`: Indentation width (default: 2)
   - `--in-place`: Modify file in-place
   - `--check`: Check if file is already formatted

3. **Batch Processing**
   - Format multiple files
   - Recursive directory formatting
   - Glob pattern support

---

## CLI Interface

```bash
# Format to pretty (default)
gbln fmt file.gbln

# Format to compact
gbln fmt --compact file.gbln

# Format in-place
gbln fmt -i file.gbln

# Custom indentation
gbln fmt --indent=4 file.gbln

# Check formatting
gbln fmt --check file.gbln
# Exit code 0: formatted correctly
# Exit code 1: needs formatting

# Multiple files
gbln fmt file1.gbln file2.gbln

# Recursive
gbln fmt --recursive src/

# Glob patterns
gbln fmt '**/*.gbln'
```

---

## File Structure

```
tools/gbln-fmt/
├── Cargo.toml
├── src/
│   ├── main.rs             # CLI entry point
│   ├── formatter.rs        # Formatting logic
│   └── options.rs          # CLI options
└── tests/
    └── format_test.rs
```

---

## Implementation

### main.rs

```rust
use clap::Parser;
use std::path::PathBuf;

#[derive(Parser)]
#[command(name = "gbln fmt")]
#[command(about = "Format GBLN files")]
struct Cli {
    /// Input files or directories
    files: Vec<PathBuf>,
    
    /// Format to compact mode
    #[arg(long)]
    compact: bool,
    
    /// Indentation width (spaces)
    #[arg(long, default_value = "2")]
    indent: usize,
    
    /// Modify files in-place
    #[arg(short, long)]
    in_place: bool,
    
    /// Check if files are formatted
    #[arg(long)]
    check: bool,
    
    /// Format directories recursively
    #[arg(short, long)]
    recursive: bool,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let cli = Cli::parse();
    
    for file in &cli.files {
        format_file(file, &cli)?;
    }
    
    Ok(())
}
```

### formatter.rs

```rust
use gbln::{parse, Value};

pub struct Formatter {
    indent: usize,
    compact: bool,
}

impl Formatter {
    pub fn new(indent: usize, compact: bool) -> Self {
        Self { indent, compact }
    }
    
    pub fn format(&self, input: &str) -> Result<String, Error> {
        let value = parse(input)?;
        
        if self.compact {
            Ok(self.format_compact(&value))
        } else {
            Ok(self.format_pretty(&value, 0))
        }
    }
    
    fn format_compact(&self, value: &Value) -> String {
        // Remove all structural whitespace
        // Keep whitespace inside () values
        todo!()
    }
    
    fn format_pretty(&self, value: &Value, depth: usize) -> String {
        // Add indentation
        // Preserve value content exactly
        todo!()
    }
}
```

---

## Examples

### Input (compact)

```gbln
user{id<u32>(12345)name<s32>(Alice)address{street<s64>(Main St)city<s16>(NYC)}}
```

### Output (pretty, indent=2)

```gbln
user{
  id<u32>(12345)
  name<s32>(Alice)
  address{
    street<s64>(Main St)
    city<s16>(NYC)
  }
}
```

### Output (pretty, indent=4)

```gbln
user{
    id<u32>(12345)
    name<s32>(Alice)
    address{
        street<s64>(Main St)
        city<s16>(NYC)
    }
}
```

---

## Edge Cases

1. **Preserve Value Whitespace**
   ```gbln
   text<s64>(Hello    World)  :| Multiple spaces preserved
   ```

2. **Comments Handling**
   - Pretty: Preserve comments
   - Compact: Option to strip or preserve

3. **Empty Objects/Arrays**
   ```gbln
   empty{}
   numbers<i32>[]
   ```

4. **Mixed Arrays**
   ```gbln
   data[
     <i32>(42)
     <s32>(hello)
   ]
   ```

---

## Testing

### Test Cases

1. **Round-trip**: compact → pretty → compact (identical)
2. **Idempotent**: pretty → pretty (identical)
3. **Preserve values**: No value content changes
4. **Edge cases**: Empty, nested, large files

### Performance

- Format 1000-record file in <100ms
- Memory usage reasonable (no full tree in memory if possible)

---

## Acceptance Criteria

- [ ] Formats compact → pretty correctly
- [ ] Formats pretty → compact correctly
- [ ] Preserves value content exactly
- [ ] Handles all edge cases
- [ ] --check mode works
- [ ] --in-place modifies files correctly
- [ ] Recursive directory formatting works
- [ ] Good error messages
- [ ] Documentation and examples

---

## Timeline

**Estimated**: 3 days  
**Dependencies**: #004 (Rust core)  
**Blocks**: None

---

## Notes

- Use gbln::parse + custom serializer
- Consider streaming for very large files
- Colorized diff output for --check mode (optional)

---

## References

- rustfmt: Inspiration for formatting tool
- prettier: Inspiration for --check mode
