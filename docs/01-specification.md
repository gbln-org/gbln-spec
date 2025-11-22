# GBLN Specification v1.0
## Formal Language Specification

**Version**: 1.0  
**Date**: 2025-01-21  
**Status**: Final  
**Authors**: Vivian Voss

---

## Table of Contents

1. [Overview](#overview)
2. [Lexical Structure](#lexical-structure)
3. [Grammar (EBNF)](#grammar-ebnf)
4. [Type System](#type-system)
5. [Data Structures](#data-structures)
6. [Validation Rules](#validation-rules)
7. [Escape Sequences](#escape-sequences)
8. [Error Handling](#error-handling)
9. [Implementation Requirements](#implementation-requirements)
10. [Conformance](#conformance)

---

## Overview

### Scope

This document defines the complete syntax, semantics, and behaviour of the GBLN (Goblin Bounded Lean Notation) serialisation format version 1.0.

**Design Goals:**
1. **LLM-optimised**: 84% fewer tokens than JSON for AI contexts
2. **Deterministic**: O(1) lookahead parsing, no ambiguity
3. **Type-safe**: Parse-time validation prevents runtime errors
4. **Memory-efficient**: Bounded types reduce RAM usage
5. **Progressive complexity**: Types optional - start fast, add safety incrementally

### Core Principles

GBLN is built on **three simple rules**:

**Rule 1: Structure**
```
record = identifier + value
value  = <type optional> + content
```

**Rule 2: Content Types**
```
(...)  → Single value (type REQUIRED)
{...}  → Object       (type NOT USED)
[...]  → Array        (type OPTIONAL, depends on items)
```

**Rule 3: Deterministic Parsing**
```
Next character after identifier/<type> determines structure:
  '(' → Single value
  '{' → Object
  '[' → Array
```

These three rules enable **deterministic parsing** and **LLM token optimisation**.

### Normative References

- **RFC 8259**: The JavaScript Object Notation (JSON) Data Interchange Format
- **Unicode Standard 15.0**: Character encoding and handling
- **IEEE 754-2019**: Floating-point arithmetic

### Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

---

## Lexical Structure

### Character Encoding

GBLN documents **MUST** be encoded in UTF-8 without BOM (Byte Order Mark).

**Valid:**
```gbln
name<s32>(Alice)
```

**Invalid:**
```
# With BOM (0xEF 0xBB 0xBF at start)
```

### Whitespace

Whitespace characters are:
- Space (U+0020)
- Horizontal Tab (U+0009)
- Line Feed (U+000A)
- Carriage Return (U+000D)

Whitespace is **OPTIONAL** between tokens except where explicitly required.

**Examples:**
```gbln
:| Minimal whitespace
user{id<u32>(123)name<s32>(Alice)}

:| With whitespace (recommended for readability)
user{
    id<u32>(123)
    name<s32>(Alice)
}
```

### Comments

Comments begin with `:| ` (colon, pipe, space) and extend to the end of the line.

**Syntax:**
```
:| comment_text
```

**Rules:**
- Comments **MAY** appear on their own line
- Comments **MAY** appear after any complete token
- Comments **MUST NOT** appear inside value literals
- The character sequence `:| ` is reserved exclusively for comments

**Examples:**
```gbln
:| Full-line comment

user{
    id<u32>(123)  :| Inline comment
    name<s32>(Alice)
}

:| Multi-line comments require multiple markers
:| Like this
:| Each line needs :|
```

**Invalid:**
```gbln
name<s32>(Ali:| commentce)  :| ERROR: Comment inside value
```

### Identifiers

Identifiers are used for object keys.

**Syntax:**
```
identifier = ( letter | "_" ) { letter | digit | "_" }
```

**Rules:**
- **MUST** start with a letter (a-z, A-Z) or underscore (_)
- **MAY** contain letters, digits (0-9), or underscores
- **MUST NOT** be empty
- Case-sensitive: `UserID` ≠ `userid`

**Valid:**
```gbln
name<s32>(Alice)
user_id<u32>(123)
_internal<u16>(42)
camelCase<s16>(value)
snake_case<s16>(value)
```

**Invalid:**
```gbln
123name<s32>(Alice)      :| ERROR: Starts with digit
user-id<u32>(123)        :| ERROR: Hyphen not allowed
<s32>(Alice)             :| ERROR: Missing identifier
```

### Reserved Sequences

The following character sequences are reserved:

| Sequence | Purpose |
|----------|---------|
| `:|` | Comment marker |
| `<` ... `>` | Type hint delimiters |
| `(` ... `)` | Value delimiters |
| `{` ... `}` | Object delimiters |
| `[` ... `]` | Array delimiters |

---

## Grammar (EBNF)

### Complete Grammar

```ebnf
(* GBLN Grammar v1.0 - Extended Backus-Naur Form *)

(* Core Concept: Every record is EITHER single-value OR multi-value *)
(* This makes parsing deterministic and unambiguous *)

(* Document Structure *)
document     = value ;
value        = object | array | key_value ;

(* Two fundamental patterns:
 * 1. Single Value Record:  identifier<type>(value)
 * 2. Multi Value Record:   identifier<type?>{...} or identifier<type?>[...]
 *)

(* Objects *)
object       = "{" , ws , [ key_values ] , ws , "}" ;
key_values   = key_value , { ws , key_value } ;

(* Key-Value Pairs *)
key_value    = identifier , "<" , type_hint , ">" , "(" , value_content , ")" ;

(* Arrays *)
array        = "[" , ws , [ array_items ] , ws , "]" ;

array_items  = typed_array 
             | mixed_array
             | object_array ;

(* Typed Array: All elements same type *)
typed_array  = identifier , "<" , type_hint , ">" , "[" , ws , [ values ] , ws , "]" ;
values       = value_content , { ws , value_content } ;

(* Mixed Array: Each element explicitly typed *)
mixed_array  = typed_value , { ws , typed_value } ;
typed_value  = "<" , type_hint , ">" , "(" , value_content , ")" ;

(* Object Array: Array of objects *)
object_array = object , { ws , object } ;

(* Value Content *)
value_content = { value_char } ;
value_char    = ( any_char - ( ")" | "\\" ) )
              | escaped_char ;

escaped_char  = "\\" , ( "\\" | "(" | ")" | "n" | "r" | "t" ) ;

(* Type Hints *)
type_hint    = int_type | float_type | string_type | bool_type | null_type ;

int_type     = ( "i" | "u" ) , ( "8" | "16" | "32" | "64" ) ;
float_type   = "f" , ( "32" | "64" ) ;
string_type  = "s" , positive_integer ;
bool_type    = "b" ;
null_type    = "n" ;

(* Identifiers *)
identifier   = ( letter | "_" ) , { letter | digit | "_" } ;

(* Basic Elements *)
letter       = "a" .. "z" | "A" .. "Z" ;
digit        = "0" .. "9" ;
positive_integer = digit , { digit } ;

(* Whitespace *)
ws           = { whitespace } ;
whitespace   = " " | "\t" | "\n" | "\r" ;

(* Comments *)
comment      = ":|" , { any_char - "\n" } , "\n" ;

(* Notes:
 * 1. Comments are preprocessed and removed before parsing
 * 2. Value content parsing tracks parenthesis depth for nested parens
 * 3. Angle brackets in value content are allowed (not delimiters there)
 *)
```

### Parsing Rules

#### 1. Comment Preprocessing

Before parsing, implementations **MUST** strip comments:

```
For each line in input:
    If line contains ":|":
        Remove everything from ":|" to end of line
    Endif
Endfor
```

#### 2. Deterministic Structure Recognition

**Every record follows one of two patterns:**

**Pattern 1: Single Value Record (Terminal)**
```
identifier<type>(value)
```

**Pattern 2: Multi Value Record (Container)**
```
identifier<type optional>{key-value pairs}   :| Object
identifier<type optional>[array items]        :| Array
```

**Parsing Algorithm:**
```
1. Read identifier
2. Read <type> (required for single values, optional for containers)
3. Peek next non-whitespace character:
   
   IF '(' → Single Value Record
      - Read until matching ')'
      - Parse value according to type
      - Return single value
   
   ELSE IF '{' → Multi Value Record (Object)
      - Recursively parse key-value pairs
      - Each pair is itself a record (single or multi)
      - Until '}'
   
   ELSE IF '[' → Multi Value Record (Array)
      - Recursively parse array items
      - Until ']'
   
   ELSE → Parse Error
```

**This structure is deterministic and unambiguous.**

#### 3. Value Content Parsing

Value content between `(` and `)` **MUST** be parsed with depth tracking:

```
depth = 0
content = ""

while current_char exists:
    if current_char == '\\':
        process_escape_sequence()
    elsif current_char == '(':
        depth += 1
        content += current_char
    elsif current_char == ')':
        if depth == 0:
            return content  # End of value
        else:
            depth -= 1
            content += current_char
        endif
    else:
        content += current_char
    endif
    advance_to_next_char()
endwhile
```

**Example:**
```gbln
formula<s64>(f(x) = (x + 1) * (x - 1))
           ^                         ^
           depth 0                   depth 0 (end)
            depth 1 ^       ^ depth 1
                     depth 2 ^ depth 2
```

#### 3. Angle Bracket Handling

After reading type hint `<type>`, angle brackets in value content are **NOT** special:

```gbln
html<s256>(<h1>Hello</h1>)
         ^                ^
         type hint        value content (< and > are literal)
```

#### 4. Whitespace Removal Rules (LLM Compression)

**Whitespace is significant ONLY inside value delimiters `()`.**

**Compression Algorithm:**
```
For each character:
    IF inside '(' ... ')':
        PRESERVE all whitespace (part of value)
    ELSE:
        REMOVE all whitespace (structural only)
```

**Examples:**

```gbln
:| Uncompressed (human-readable)
user{
    id<u32>(12345)
    name<s64>(Alice Johnson)
}

:| Compressed (LLM-optimised) - EQUIVALENT
user{id<u32>(12345)name<s64>(Alice Johnson)}
```

**Key Point:** `(Alice Johnson)` preserves the space, but whitespace between `}` and `name` is structural and can be removed.

**Decompression:**
```
The formatter can add whitespace back for human-readability
without changing semantics, as long as value content is preserved.
```

---

## Type System

### Overview

GBLN provides **strongly-typed**, **bounded** data types. Every value **MUST** have an explicit type hint.

### Integer Types

#### Signed Integers

| Type | Minimum Value | Maximum Value | Memory | Typical Use |
|------|---------------|---------------|--------|-------------|
| `i8` | -128 | 127 | 1 byte | Age, small counters |
| `i16` | -32,768 | 32,767 | 2 bytes | Coordinates, port numbers |
| `i32` | -2,147,483,648 | 2,147,483,647 | 4 bytes | IDs, timestamps |
| `i64` | -9,223,372,036,854,775,808 | 9,223,372,036,854,775,807 | 8 bytes | Large numbers |

**Syntax:**
```gbln
age<i8>(25)
port<i16>(8080)
timestamp<i64>(1737475200)
```

**Validation:**
- Value **MUST** parse as a decimal integer
- Value **MUST** fit within the type's range
- Leading zeros are allowed: `age<i8>(025)` = 25
- Sign **MUST** be `-` for negative, none or `+` for positive
- Scientific notation **NOT** allowed

**Valid:**
```gbln
age<i8>(25)
age<i8>(-5)
age<i8>(+25)
age<i8>(025)   :| Leading zeros OK
```

**Invalid:**
```gbln
age<i8>(999)     :| ERROR: Out of range
age<i8>(25.5)    :| ERROR: Decimal point not allowed
age<i8>(1e2)     :| ERROR: Scientific notation not allowed
```

#### Unsigned Integers

| Type | Minimum Value | Maximum Value | Memory | Typical Use |
|------|---------------|---------------|--------|-------------|
| `u8` | 0 | 255 | 1 byte | Percentages, status codes |
| `u16` | 0 | 65,535 | 2 bytes | Port numbers, counts |
| `u32` | 0 | 4,294,967,295 | 4 bytes | User IDs, hashes |
| `u64` | 0 | 18,446,744,073,709,551,615 | 8 bytes | Large counters, file sizes |

**Syntax:**
```gbln
status<u8>(200)
user_id<u32>(12345)
file_size<u64>(1099511627776)
```

**Validation:**
- Value **MUST** parse as a decimal integer
- Value **MUST** be non-negative
- Value **MUST** fit within the type's range
- Sign **MUST NOT** be negative

**Valid:**
```gbln
count<u8>(200)
count<u8>(+200)  :| Explicit + allowed
count<u8>(0)
```

**Invalid:**
```gbln
count<u8>(-5)    :| ERROR: Negative value for unsigned type
count<u8>(300)   :| ERROR: Out of range (max 255)
```

### Float Types

| Type | Format | Precision | Memory | Typical Use |
|------|--------|-----------|--------|-------------|
| `f32` | IEEE 754 single | ~7 decimal digits | 4 bytes | Prices, percentages |
| `f64` | IEEE 754 double | ~15 decimal digits | 8 bytes | Scientific data, coordinates |

**Syntax:**
```gbln
price<f32>(19.99)
latitude<f64>(51.5074)
scientific<f64>(1.23e-10)
```

**Validation:**
- Value **MUST** parse as a floating-point number
- IEEE 754 special values allowed: `inf`, `-inf`, `nan`

**Valid:**
```gbln
price<f32>(19.99)
price<f32>(19)       :| Decimal point optional
price<f32>(.5)       :| Leading digit optional
price<f32>(1.23e10)  :| Scientific notation OK
inf<f32>(inf)        :| Infinity
nan<f32>(nan)        :| Not a Number
```

**Invalid:**
```gbln
price<f32>(abc)      :| ERROR: Not a number
```

### String Types

String types specify **maximum character count** (not byte count).

| Type | Max Characters | Typical Use |
|------|----------------|-------------|
| `s2` | 2 | Country codes (US, DE) |
| `s4` | 4 | Short codes |
| `s8` | 8 | Status codes, short tags |
| `s16` | 16 | Usernames, short tags |
| `s32` | 32 | Names, short descriptions |
| `s64` | 64 | Emails, URLs |
| `s128` | 128 | Longer URLs, text |
| `s256` | 256 | Descriptions, paragraphs |
| `s512` | 512 | Long content |
| `s1024` | 1024 | Very long content |

**Syntax:**
```gbln
country<s2>(DE)
username<s16>(alice_dev)
email<s64>(alice@example.com)
```

**Validation:**
- Character count **MUST** be counted using UTF-8 character boundaries
- Character count **MUST NOT** exceed the specified maximum
- Empty strings are allowed: `name<s32>()`
- Leading/trailing whitespace is **PRESERVED**

**UTF-8 Character Counting:**

```gbln
:| ASCII: 1 byte per character
name<s5>(Hello)           :| 5 characters, 5 bytes ✅

:| Multi-byte characters count as 1 character
city<s2>(北京)            :| 2 characters, 6 bytes ✅

:| Emoji count as 1 character
greeting<s6>(Hello🔥)     :| 6 characters, 10 bytes ✅

:| Combining characters count separately
e_acute<s2>(é)            :| 1 or 2 chars depending on normalisation
```

**Valid:**
```gbln
name<s32>(Alice Johnson)
empty<s32>()                    :| Empty string OK
spaces<s16>(   )                :| Whitespace preserved
```

**Invalid:**
```gbln
name<s8>(VeryLongNameHere)      :| ERROR: 17 chars > 8
```

### Boolean Type

| Type | Valid Values | Description |
|------|-------------|-------------|
| `b` | `t`, `f`, `true`, `false`, `1`, `0` | Boolean value |

**Syntax:**
```gbln
active<b>(t)
verified<b>(true)
debug<b>(0)
```

**Validation:**
- Value **MUST** be one of: `t`, `f`, `true`, `false`, `1`, `0`
- Comparison is **case-insensitive**
- Whitespace is trimmed before comparison

**Valid:**
```gbln
active<b>(t)
active<b>(f)
active<b>(true)
active<b>(false)
active<b>(True)      :| Case-insensitive
active<b>(FALSE)
active<b>(1)
active<b>(0)
```

**Invalid:**
```gbln
active<b>(yes)       :| ERROR: Use t/f/true/false/0/1
active<b>(no)
active<b>(2)
active<b>()          :| ERROR: Empty not allowed
```

### Null Type

| Type | Valid Values | Description |
|------|-------------|-------------|
| `n` | empty, `n`, `null` | Null/absent value |

**Syntax:**
```gbln
optional<n>()
optional<n>(null)
optional<n>(n)
```

**Validation:**
- Value **MUST** be empty, `n`, or `null`
- Comparison is **case-insensitive**
- Whitespace is trimmed

**Valid:**
```gbln
optional<n>()
optional<n>(null)
optional<n>(n)
optional<n>(NULL)    :| Case-insensitive
optional<n>(Null)
```

**Invalid:**
```gbln
optional<n>(nil)     :| ERROR: Use null/n or empty
```

---

## Data Structures

### Objects

Objects are collections of key-value pairs.

**Syntax:**
```gbln
object_name{
    key1<type1>(value1)
    key2<type2>(value2)
}
```

**Rules:**
- Keys **MUST** be unique within an object
- Key order **MUST** be preserved (insertion order)
- Empty objects are allowed: `empty{}`
- Whitespace between key-value pairs is **OPTIONAL** but **RECOMMENDED**

**Valid:**
```gbln
user{
    id<u32>(123)
    name<s32>(Alice)
}

:| Minimal (no whitespace)
user{id<u32>(123)name<s32>(Alice)}

:| Empty
empty{}
```

**Invalid:**
```gbln
user{
    id<u32>(123)
    id<u32>(456)     :| ERROR: Duplicate key 'id'
}
```

**Nested Objects:**
```gbln
company{
    name<s64>(TechCorp)
    
    address{
        street<s64>(123 Main St)
        city<s32>(NYC)
    }
}
```

**Nesting Depth:**
- Implementations **SHOULD** support at least 64 levels of nesting
- Implementations **MAY** reject deeper nesting

### Homogeneous Arrays

Arrays where all elements have the **same type**.

**Syntax:**
```gbln
array_name<type>[value1 value2 value3]
```

**Rules:**
- Type specified once at array level
- All elements **MUST** be parseable as the specified type
- Elements separated by whitespace (not commas)
- Empty arrays allowed: `numbers<i32>[]`

**Valid:**
```gbln
:| Integers
numbers<i32>[1 2 3 4 5]

:| Strings
tags<s16>[rust python golang javascript]

:| Floats
temperatures<f32>[18.5 19.2 22.4 23.1]

:| Empty
empty<i32>[]

:| Multi-line
numbers<i32>[
    1
    2
    3
]
```

**Invalid:**
```gbln
:| Strings with spaces need object arrays or mixed arrays
tags<s16>[hello world]   :| ERROR: Parsed as ["hello", "world"]
                          :| Use tags[<s16>(hello world)] instead
```

### Mixed Arrays

Arrays where elements have **different types**.

**Syntax:**
```gbln
array_name[
    <type1>(value1)
    <type2>(value2)
    <type3>(value3)
]
```

**Rules:**
- Each element **MUST** have explicit type hint
- Elements can be any type (including objects and arrays)
- Empty arrays allowed: `mixed[]`

**Valid:**
```gbln
data[
    <i32>(42)
    <s64>(hello world)
    <b>(t)
    <f32>(3.14)
]

:| Mixed with objects
items[
    <s32>(text)
    {id<u32>(1) name<s32>(Alice)}
    <i32>(42)
]
```

### Object Arrays

Arrays where all elements are **objects**.

**Syntax:**
```gbln
array_name[
    {key1<type>(val1) key2<type>(val2)}
    {key1<type>(val1) key2<type>(val2)}
]
```

**Rules:**
- Objects don't need type prefix
- Objects **MAY** have different structures (not enforced)
- **RECOMMENDED**: Keep same structure for clarity

**Valid:**
```gbln
users[
    {id<u32>(1) name<s32>(Alice) age<i8>(25)}
    {id<u32>(2) name<s32>(Bob) age<i8>(30)}
]

:| Different structures (allowed but not recommended)
mixed_objects[
    {id<u32>(1) name<s32>(Alice)}
    {type<s16>(admin) level<u8>(5)}
]
```

---

## Validation Rules

### Parse-Time Validation

All validation **MUST** occur during parsing. Invalid data **MUST NOT** be accepted.

#### Rule 1: Integer Range Validation

**Requirement:**
- Integer values **MUST** fit within the specified type's range
- Parsers **MUST** reject out-of-range values

**Example:**
```gbln
age<i8>(25)      :| OK: 25 ∈ [-128, 127]
age<i8>(999)     :| ERROR: 999 ∉ [-128, 127]
count<u8>(200)   :| OK: 200 ∈ [0, 255]
count<u8>(300)   :| ERROR: 300 ∉ [0, 255]
```

**Error Format:**
```
Error: Integer out of range
  at field: age
  value: 999
  type: i8
  valid range: -128 to 127
  line: 5
  column: 15
  
  suggestion: Use i16 or i32 for larger values
```

#### Rule 2: String Length Validation

**Requirement:**
- String character count **MUST NOT** exceed the specified maximum
- Character count **MUST** use UTF-8 character boundaries
- Empty strings are valid

**Example:**
```gbln
name<s32>(Alice)                    :| OK: 5 chars ≤ 32
email<s64>(alice@example.com)       :| OK: 19 chars ≤ 64
code<s8>(VeryLongCode)              :| ERROR: 12 chars > 8
```

**Error Format:**
```
Error: String exceeds maximum length
  at field: code
  value: "VeryLongCode"
  actual: 12 characters
  maximum: 8 characters (s8)
  line: 3
  column: 10
  
  suggestion: Use s16 or s32 for longer strings
```

#### Rule 3: Type Match Validation

**Requirement:**
- Value **MUST** be parseable as the specified type
- Type conversion **MUST NOT** be automatic

**Example:**
```gbln
age<i8>(25)          :| OK: "25" parses as integer
price<f32>(19.99)    :| OK: "19.99" parses as float
active<b>(t)         :| OK: "t" is valid boolean

age<i8>(abc)         :| ERROR: "abc" not parseable as integer
price<f32>(invalid)  :| ERROR: "invalid" not parseable as float
count<u32>(3.14)     :| ERROR: "3.14" has decimal (not integer)
```

**Error Format:**
```
Error: Type validation failed
  at field: age
  expected: integer (i8)
  received: "abc"
  line: 4
  column: 20
  
  suggestion: Ensure value is a valid integer
```

#### Rule 4: Duplicate Key Validation

**Requirement:**
- Object keys **MUST** be unique within their scope
- Duplicate keys **MUST** cause parse error

**Example:**
```gbln
:| Invalid
user{
    id<u32>(1)
    name<s32>(Alice)
    id<u32>(2)        :| ERROR: Duplicate key 'id'
}
```

**Error Format:**
```
Error: Duplicate key in object
  key: "id"
  first occurrence: line 2, column 5
  duplicate: line 4, column 5
  
  suggestion: Remove duplicate key or rename one of them
```

#### Rule 5: Boolean Value Validation

**Requirement:**
- Boolean values **MUST** use recognised representations
- Case-insensitive matching

**Example:**
```gbln
active<b>(t)         :| OK
active<b>(true)      :| OK
active<b>(1)         :| OK

active<b>(yes)       :| ERROR: Use t/f/true/false/0/1
active<b>(2)         :| ERROR: Use 0 or 1
```

---

## Escape Sequences

### Supported Escapes

Escape sequences **MUST** be supported within value content.

| Sequence | Result | Unicode | Description |
|----------|--------|---------|-------------|
| `\\` | `\` | U+005C | Backslash |
| `\n` | (newline) | U+000A | Line Feed |
| `\r` | (carriage return) | U+000D | Carriage Return |
| `\t` | (tab) | U+0009 | Horizontal Tab |
| `\(` | `(` | U+0028 | Left Parenthesis |
| `\)` | `)` | U+0029 | Right Parenthesis |

**Examples:**
```gbln
path<s128>(C:\\Users\\Alice\\Documents)
multiline<s256>(Line 1\nLine 2\nLine 3)
formatted<s64>(Name:\tAlice\tAge:\t25)
parens<s32>(Formula: \(x + 1\) = 5)
```

### Characters NOT Requiring Escape

The following do **NOT** require escaping in value content:

- **Angle brackets** `<` `>` - parsed as literal characters
- **Braces** `{` `}` - parsed as literal characters
- **Brackets** `[` `]` - parsed as literal characters
- **Nested parentheses** - handled via depth tracking

**Examples:**
```gbln
html<s256>(<h1>Hello World</h1>)
xml<s512>(<user><name>Alice</name></user>)
generic<s64>(Vec<HashMap<String, Value>>)
formula<s64>(f(x) = (x + 1) * (x - 1))
```

### Invalid Escape Sequences

Escape sequences **NOT** in the table above **SHOULD** be preserved as-is:

```gbln
text<s32>(unknown: \x41)  :| Preserved as "unknown: \x41"
```

Implementations **MAY** warn about unknown escape sequences but **MUST NOT** reject them.

---

## Error Handling

### Error Message Format

Implementations **MUST** provide detailed error messages in the following format:

```
Error: <error_category>
  at field: <field_path>
  <context-specific details>
  line: <line_number>
  column: <column_number>
  
  suggestion: <helpful_suggestion>
```

### Error Categories

#### Syntax Errors

**Unexpected Token:**
```
Error: Unexpected token
  expected: ')'
  found: ']'
  line: 10
  column: 30
  
  suggestion: Check for matching parentheses
```

**Unexpected End of Input:**
```
Error: Unexpected end of input
  expected: closing '}'
  context: object started at line 5
  line: 15
  
  suggestion: Add closing brace for object
```

#### Type Errors

**Integer Out of Range:**
```
Error: Integer out of range
  at field: user.age
  value: 999
  type: i8
  valid range: -128 to 127
  line: 8
  column: 18
  
  suggestion: Use i16 (range: -32768 to 32767)
```

**String Too Long:**
```
Error: String exceeds maximum length
  at field: user.name
  value: "VeryLongNameThatExceedsLimit"
  actual: 28 characters
  maximum: 16 characters (s16)
  line: 6
  column: 12
  
  suggestion: Use s32 or s64 for longer strings
```

**Type Mismatch:**
```
Error: Type validation failed
  at field: settings.port
  expected: unsigned integer (u16)
  received: "abc" (not parseable as integer)
  line: 12
  column: 25
  
  suggestion: Ensure value is a valid unsigned integer (0-65535)
```

#### Structural Errors

**Duplicate Key:**
```
Error: Duplicate key in object
  key: "id"
  first occurrence: line 3, column 5
  duplicate: line 7, column 5
  
  suggestion: Remove duplicate key or rename one of them
```

### Line and Column Tracking

Implementations **MUST** track line and column numbers:

- **Lines**: Start at 1
- **Columns**: Start at 1
- **Line breaks**: LF (U+000A), CR (U+000D), or CRLF (U+000D U+000A)

---

## Implementation Requirements

### MUST Support

Conforming implementations **MUST** support:

- ✅ All integer types (i8-i64, u8-u64)
- ✅ All float types (f32, f64)
- ✅ All string types (s2, s4, s8, s16, s32, s64, s128, s256, s512, s1024)
- ✅ Boolean type (b)
- ✅ Null type (n)
- ✅ Objects with nested structures
- ✅ Homogeneous arrays
- ✅ Mixed arrays
- ✅ Object arrays
- ✅ Comments (:|)
- ✅ All escape sequences
- ✅ Parse-time validation
- ✅ Detailed error messages with line/column numbers
- ✅ UTF-8 encoding
- ✅ UTF-8 aware string length counting

### SHOULD Support

Conforming implementations **SHOULD** support:

- ⚠️ Streaming parsing (for large files)
- ⚠️ Pretty printing (formatting output)
- ⚠️ Round-trip guarantee (parse → serialise → parse)
- ⚠️ At least 64 levels of nesting

### MAY Support

Conforming implementations **MAY** support:

- ⭕ Schema validation (external schema files)
- ⭕ Custom type extensions
- ⭕ Binary serialisation format
- ⭕ Compression

---

## Conformance

### Conformance Levels

**Level 1: Basic Conformance**
- Supports all MUST requirements
- Rejects invalid input with error messages
- Preserves data integrity

**Level 2: Full Conformance**
- Level 1 + all SHOULD requirements
- Round-trip guarantee
- Pretty printing support

**Level 3: Extended Conformance**
- Level 2 + optional features
- Schema validation
- Additional tooling

### Test Suite

A normative test suite **SHALL** be provided containing:

1. **Valid Inputs**: MUST parse successfully
2. **Invalid Inputs**: MUST reject with specific errors
3. **Edge Cases**: UTF-8, nesting, escaping
4. **Round-Trip**: parse(serialise(x)) == x

---

## Appendix A: Type Range Reference

### Integer Ranges

```
i8:  -128 to 127
i16: -32768 to 32767
i32: -2147483648 to 2147483647
i64: -9223372036854775808 to 9223372036854775807

u8:  0 to 255
u16: 0 to 65535
u32: 0 to 4294967295
u64: 0 to 18446744073709551615
```

### String Character Limits

```
s2:    2 characters
s4:    4 characters
s8:    8 characters
s16:   16 characters
s32:   32 characters
s64:   64 characters
s128:  128 characters
s256:  256 characters
s512:  512 characters
s1024: 1024 characters
```

---

## Appendix B: MIME Type

**Proposed MIME Type:** `application/gbln`  
**File Extension:** `.gbln`

---

## Appendix C: Version History

### Version 1.0 (2025-01-21)
- Initial specification release
- Complete type system
- Formal grammar (EBNF)
- Validation rules
- Error handling specification

---

*GBLN Specification v1.0*  
*© 2025 Vivian Voss - Apache 2.0 License*

---

**End of Specification**
