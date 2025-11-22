# Real Benchmark Results - Employee Data (5 Records)

## Test Dataset

- **File**: `employees.*`
- **Records**: 5 employees with complete HR information
- **Complexity**: Nested structures (address object, certifications array of objects)
- **Data types**: Strings, numbers, booleans, dates, arrays, nested objects

## File Size Comparison

| Format | Bytes | vs JSON | vs Smallest | Notes |
|--------|-------|---------|-------------|-------|
| **GBLN (compressed)** | **3,138** | **-29%** | **baseline** ⭐ | Libraries auto-compress for transmission |
| TOON | 3,326 | -25% | +6% | Falls back to YAML-style for nested data |
| YAML | 3,535 | -20% | +13% | Standard indentation format |
| JSON | 4,431 | baseline | +41% | Universal standard |
| GBLN (human-readable) | 4,757 | +7% | +52% | With formatting/indentation |

**Winner: GBLN (compressed)** - 6% smaller than TOON, 11% smaller than YAML, 29% smaller than JSON.

**IMPORTANT**: GBLN libraries automatically send compressed format. The readable format is only for human editing.

## Token Estimation (word count approximation)

| Format | Words | vs JSON | vs Fewest |
|--------|-------|---------|-----------|
| **GBLN (compressed)** | **47** | **-86%** ⭐ | **baseline** |
| GBLN (human-readable) | 202 | -38% | +330% |
| TOON | 280 | -15% | +496% |
| YAML | 315 | -4% | +570% |
| JSON | 328 | baseline | +598% |

**Note**: Word count (`wc -w`) is NOT accurate for LLM tokens. Proper measurement requires tiktoken or GPT-4 tokenizer.

**Winner: GBLN (compressed)** - 83% fewer "words" than TOON, 86% fewer than JSON.

## Key Findings

### Why GBLN Compressed Wins

1. **Whitespace Removal**: All structural whitespace outside `()` delimiters is removable
2. **No Redundancy**: Type hints are concise (`<s32>`, `<u16>`) vs YAML/TOON indentation
3. **Nested Data**: Handles complexity without falling back to verbose formats

### Why TOON Doesn't Win Here

TOON's strength is **uniform arrays with primitive values** (CSV-style tables). For this dataset:
- ❌ Nested `address` objects → can't use tabular format
- ❌ Array of `certifications` objects → can't use tabular format
- ❌ Falls back to YAML-style indentation → loses CSV compression advantage

**From TOON docs**: "TOON's sweet spot is uniform arrays of objects (multiple fields per row, same structure across items). Deeply nested or non-uniform structures have minimal tabular eligibility."

### Format-Specific Observations

**JSON (4,431 bytes)**:
- Verbose: Repeated field names, quoted keys, brackets everywhere
- Universal support, but wasteful for data transfer

**YAML (3,535 bytes)**:
- Saves ~20% vs JSON through indentation instead of brackets
- Human-friendly but indentation-sensitive

**TOON (3,326 bytes)**:
- Slightly better than YAML (6% smaller)
- Could use `skills[4]: value,value` for primitive arrays
- But nested objects force YAML-style fallback

**GBLN compressed (3,138 bytes)**:
- Smallest of all formats (6% smaller than TOON)
- Type hints included: `firstName<s32>(Alice)` vs YAML `firstName: Alice`
- Whitespace completely removed except within `()` value delimiters

**GBLN readable (4,757 bytes)**:
- Larger than JSON due to type hints + formatting
- But provides parse-time validation and memory bounds
- Human-editable with inline type information

## The TOON vs GBLN Trade-Off

### When TOON Would Win

TOON excels with **flat, uniform data**:

```toon
employees[1000]{id,firstName,lastName,email,salary}:
  1,Alice,Johnson,alice@company.com,75000
  2,Bob,Schmidt,bob@company.com,68000
  ...
```

Schema declared once, values streamed as CSV rows. Very compact!

### When GBLN Wins (This Dataset)

GBLN excels with **nested, complex data**:

```gbln
employees[
    {
        id<u16>(1)
        firstName<s32>(Alice)
        address{street<s64>(Hauptstraße 123)city<s32>(Berlin)}
        certifications[{name<s64>(AWS Certified)issueDate<s16>(2020-05-10)}]
    }
]
```

Compresses completely while preserving type safety and structure.

## Real-World Implications

### For LLM Token Costs (Cloud APIs)

**Small context (< 10KB)**:
- All formats are cheap enough (< $0.01 per request)
- Token savings matter less than reliability

**Large context (> 100KB)**:
- Use vector search/RAG instead of dumping data
- Token optimisation is wrong solution to architecture problem

### For Production Systems

**Type Safety**:
- ✅ GBLN: Parse-time validation (`id<u16>` validates 0-65535)
- ❌ JSON/YAML/TOON: No validation, runtime errors

**Memory Bounds**:
- ✅ GBLN: Bounded strings (`s32` = max 32 chars)
- ❌ JSON/YAML/TOON: Unbounded, potential OOM

**Deterministic Parsing**:
- ✅ GBLN: O(1) lookahead, 3 simple rules
- ⚠️ TOON: Schema-dependent, indentation-sensitive
- ❌ YAML: Complex spec, many gotchas

**LLM Output Validation**:
- ✅ GBLN: Catches hallucinated data at parse-time
- ❌ Others: Accept invalid data, crash at runtime

## Conclusion

**Different formats for different use cases:**

### Use TOON when:
- Data is **flat and uniform** (like CSV tables)
- Minimizing tokens for **cloud API costs** is critical
- No type validation needed

### Use GBLN when:
- Data has **nested structures** (this benchmark)
- **Type safety and memory bounds** matter (production systems)
- **LLM output validation** needed (catch hallucinations)
- **Deterministic parsing** required (3 simple rules)
- Smallest possible **compressed size** desired

**For this specific employee dataset**: GBLN compressed is the clear winner (smallest + type-safe).

---

## Benchmark Files

All files use **valid, optimised syntax** for their respective formats:

- `employees.json` - Standard JSON
- `employees.yaml` - Standard YAML
- `employees.toon` - Valid TOON (YAML-style for nested data, arrays where possible)
- `employees.gbln` - GBLN with formatting (human-readable)
- `employees-compressed.gbln` - GBLN compressed (what libraries send)

---

**Last Updated**: 2025-01-22  
**Dataset**: 5 employee records with nested address and certifications
