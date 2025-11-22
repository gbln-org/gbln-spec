# GBLN Specification v1.1

**Version**: 1.1  
**Date**: 2025-01-20  
**Status**: Final - Ready for Implementation

## Table of Contents

1. [Core Syntax](#core-syntax)
2. [Type System](#type-system)
3. [Data Structures](#data-structures)
4. [Validation Rules](#validation-rules)
5. [Escaping](#escaping)
6. [Grammar (EBNF)](#grammar-ebnf)
7. [Error Handling](#error-handling)

---

## Core Syntax

### Basic Structure

```gbln
key<type>(value)
```

**Components:**
- `key`: Identifier (letters, digits, underscore, must start with letter or underscore)
- `<type>`: Type hint in angle brackets
- `(value)`: Value in parentheses

**Examples:**
```gbln
name<s64>(Alice Johnson)
age<i8>(25)
price<f32>(19.99)
active<b>(t)
_internal<u32>(12345)
user_id<u32>(67890)
```

### Comments

```gbln
:| This is a comment
```

**Rules:**
- Single-line only
- Everything after `:|` until newline is ignored
- Can appear anywhere except inside value literals
- Unique to GBLN (no other format uses `:|`)

**Examples:**
```gbln
:| Full-line comment

user{
    id<u32>(12345)     :| Inline comment
    name<s64>(Alice)   :| Another inline comment
}

:| Multi-line comments require multiple comment markers
:| Like this
:| Each line needs :|
```

---

## Type System

### Integers (bit-width)

#### Signed Integers

| Type | Range | Memory | Use Case |
|------|-------|--------|----------|
| `i8` | -128 to 127 | 1 byte | Age, small counters, percentages |
| `i16` | -32,768 to 32,767 | 2 bytes | Port numbers, coordinates |
| `i32` | -2,147,483,648 to 2,147,483,647 | 4 bytes | IDs, timestamps, counters |
| `i64` | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 | 8 bytes | Large numbers, Unix timestamps |

#### Unsigned Integers

| Type | Range | Memory | Use Case |
|------|-------|--------|----------|
| `u8` | 0 to 255 | 1 byte | Percentages, status codes, flags |
| `u16` | 0 to 65,535 | 2 bytes | Port numbers, counts |
| `u32` | 0 to 4,294,967,295 | 4 bytes | User IDs, counters, hashes |
| `u64` | 0 to 18,446,744,073,709,551,615 | 8 bytes | Large counters, hashes, file sizes |

**Integer Parsing Rules:**
- Leading zeros allowed: `age<i8>(025)` = 25
- Sign required for negatives: `temp<i8>(-5)`
- No sign or `+` for positives: `count<u8>(100)` or `count<u8>(+100)`
- No decimal points: `count<u32>(3.14)` is ERROR
- No scientific notation: `big<u64>(1e9)` is ERROR
- Whitespace trimmed: `age<i8>( 25 )` = 25

### Floats (precision)

| Type | Format | Precision | Memory | Use Case |
|------|--------|-----------|--------|----------|
| `f32` | IEEE 754 single | ~7 decimal digits | 4 bytes | Prices, percentages, coordinates |
| `f64` | IEEE 754 double | ~15 decimal digits | 8 bytes | Scientific data, precise coordinates |

**Float Parsing Rules:**
- Decimal point optional: `price<f32>(19)` = 19.0
- Scientific notation supported: `big<f64>(1.23e10)`
- Leading/trailing zeros: `val<f32>(0.5)` or `val<f32>(.5)`
- Infinity: `inf<f32>(inf)` or `inf<f32>(-inf)`
- Not a Number: `nan<f32>(nan)` or `nan<f32>(NaN)`
- Whitespace trimmed

### Strings (character limit)

| Type | Max Characters | Description | Example Use Case |
|------|---------------|-------------|------------------|
| `s2` | 2 | Country codes | `country<s2>(US)` |
| `s4` | 4 | Short codes | `code<s4>(ABCD)` |
| `s8` | 8 | Short codes, status | `status<s8>(active)` |
| `s16` | 16 | Usernames, tags | `username<s16>(alice_dev)` |
| `s32` | 32 | Names, short descriptions | `name<s32>(Alice Johnson)` |
| `s64` | 64 | Emails, longer text | `email<s64>(alice@example.com)` |
| `s128` | 128 | URLs, long text | `url<s128>(https://example.com/path)` |
| `s256` | 256 | Descriptions, paragraphs | `desc<s256>(Long description...)` |
| `s512` | 512 | Long content | `content<s512>(Very long text...)` |
| `s1024` | 1024 | Very long content | `article<s1024>(Article text...)` |

**Important Notes:**
- Character count, NOT byte count (UTF-8 aware)
- Empty strings allowed: `name<s32>()`
- Leading/trailing whitespace preserved
- No automatic trimming

**Examples:**
```gbln
country<s2>(DE)                  :| 2 characters
username<s16>(alice_dev)         :| 9 characters
email<s64>(alice@example.com)    :| 19 characters
city<s4>(北京)                   :| 2 Chinese characters (6 bytes in UTF-8)
emoji<s8>(Hello🔥)               :| 6 characters (5 ASCII + 1 emoji)
empty<s32>()                     :| 0 characters - valid
spaces<s16>(   )                 :| 3 characters - whitespace preserved
```

### Boolean

| Type | Values | Description |
|------|--------|-------------|
| `b` | `t`, `f`, `true`, `false`, `1`, `0` | Boolean |

**Boolean Parsing Rules:**
- Case-insensitive: `active<b>(True)` = true
- Whitespace trimmed: `active<b>( t )` = true
- Any other value is ERROR

**Examples:**
```gbln
:| All valid:
active<b>(t)
active<b>(f)
active<b>(true)
active<b>(false)
active<b>(True)
active<b>(FALSE)
active<b>(1)
active<b>(0)

:| Invalid:
active<b>(yes)    :| ERROR
active<b>(no)     :| ERROR
active<b>(2)      :| ERROR
active<b>()       :| ERROR
```

### Null

| Type | Values | Description |
|------|--------|-------------|
| `n` | empty, `n`, `null` | Null/empty value |

**Null Parsing Rules:**
- Empty parens: `value<n>()`
- Explicit null: `value<n>(null)` or `value<n>(n)`
- Case-insensitive: `value<n>(NULL)` or `value<n>(Null)`
- Whitespace trimmed

**Examples:**
```gbln
:| All represent null:
optional<n>()
optional<n>(null)
optional<n>(n)
optional<n>(NULL)
optional<n>(Null)
```

---

## Data Structures

### Objects

```gbln
user{
    id<u32>(12345)
    name<s64>(Alice Johnson)
    age<i8>(25)
    active<b>(t)
}
```

**Rules:**
- Keys must be unique within object (parse error on duplicate)
- Keys preserve insertion order (deterministic iteration)
- Unlimited nesting depth (implementation-dependent, recommend 64 max)
- Whitespace between key-value pairs is optional but recommended
- Empty objects allowed: `empty{}`
- Trailing whitespace allowed

**Examples:**
```gbln
:| Minimal
user{id<u32>(1) name<s32>(Alice)}

:| With newlines
user{
    id<u32>(1)
    name<s32>(Alice)
}

:| Empty
empty{}

:| Nested
company{
    name<s64>(TechCorp)
    address{
        street<s64>(123 Main St)
        city<s32>(NYC)
    }
}
```

### Homogeneous Arrays

```gbln
:| All items same type
numbers<i32>[1 2 3 4 5]
tags<s16>[rust python go javascript]
temperatures<f32>[18.5 19.2 22.4 23.1]
```

**Format:** `key<type>[item1 item2 item3]`

**Rules:**
- Type specified at array level
- All items must be parseable as specified type
- Space-separated items
- No commas between items
- Empty arrays allowed: `numbers<i32>[]`
- Newlines allowed between items

**Examples:**
```gbln
:| Single line
numbers<i32>[1 2 3]

:| Multi-line
numbers<i32>[
    1
    2
    3
]

:| Empty
empty<i32>[]

:| Strings with spaces must be in objects/mixed arrays
:| This is INVALID: tags<s16>[hello world]  (parsed as two items)
```

### Mixed Arrays

```gbln
:| Each item explicitly typed
mixed[
    <i32>(42)
    <s16>(hello)
    <b>(t)
    <f32>(3.14)
]
```

**Format:** Each item is `<type>(value)`

**Rules:**
- Use when array contains different types
- Each item has inline type annotation
- More verbose but type-safe
- Empty arrays allowed: `mixed[]`
- Can mix any types including objects and arrays

**Examples:**
```gbln
:| Mixed primitives
data[
    <i32>(123)
    <s64>(hello world)
    <f32>(3.14)
    <b>(t)
]

:| Mixed with objects
items[
    <s32>(text)
    {id<u32>(1) name<s32>(Alice)}
    <i32>(42)
]
```

### Array of Objects

```gbln
users[
    {id<u32>(1) name<s32>(Alice) age<i8>(25)}
    {id<u32>(2) name<s32>(Bob) age<i8>(30)}
    {id<u32>(3) name<s32>(Charlie) age<i8>(35)}
]
```

**Rules:**
- Objects don't need type prefix in object arrays
- All objects can have different structures
- Recommended: keep same structure for clarity
- Empty array allowed: `users[]`

### Nested Structures

```gbln
company{
    name<s64>(Tech Corp)
    
    address{
        street<s64>(123 Main St)
        city<s32>(San Francisco)
        zip<s16>(94102)
    }
    
    employees[
        {id<u32>(1) name<s32>(Alice)}
        {id<u32>(2) name<s32>(Bob)}
    ]
    
    metadata{
        tags<s16>[tech startup saas]
        ratings<f32>[4.5 4.8 4.9]
    }
}
```

---

## Validation Rules

### Parse-Time Validation

GBLN validates all values at parse-time, preventing invalid data from entering the system.

#### 1. Integer Range Validation

Values must fit within the specified integer type's range.

```gbln
:| Valid
age<i8>(25)          :| OK: 25 is in [-128, 127]
count<u8>(200)       :| OK: 200 is in [0, 255]

:| Invalid
age<i8>(999)         :| ERROR: 999 out of range [-128, 127]
count<u8>(-5)        :| ERROR: -5 out of range [0, 255]
port<u16>(70000)     :| ERROR: 70000 out of range [0, 65535]
```

**Error Format:**
```
Error: Integer out of range
  at field: age
  value: 999
  type: i8
  valid range: -128 to 127
  line: 3
  column: 15
  
  suggestion: Use i16 or i32 for larger values
```

#### 2. String Length Validation

Character count must not exceed the specified limit.

```gbln
:| Valid
name<s32>(Alice)                    :| OK: 5 chars <= 32
email<s64>(alice@example.com)       :| OK: 19 chars <= 64

:| Invalid
name<s8>(VeryLongNameHere)          :| ERROR: 17 chars > 8
code<s2>(USA)                       :| ERROR: 3 chars > 2
```

**Error Format:**
```
Error: String exceeds maximum length
  at field: name
  value: "VeryLongNameHere"
  actual: 17 characters
  maximum: 8 characters (s8)
  line: 2
  column: 10
  
  suggestion: Use s32 or s64 for longer strings
```

#### 3. Type Match Validation

Value must be parseable as the specified type.

```gbln
:| Valid
age<i8>(25)          :| OK: "25" parses as integer
price<f32>(19.99)    :| OK: "19.99" parses as float
active<b>(t)         :| OK: "t" is valid boolean

:| Invalid
age<i8>(abc)         :| ERROR: "abc" is not a valid integer
price<f32>(invalid)  :| ERROR: "invalid" is not a valid float
count<u32>(3.14)     :| ERROR: "3.14" contains decimal point (not integer)
```

**Error Format:**
```
Error: Type validation failed
  at field: age
  expected: integer (i8)
  received: "abc" (not parseable as integer)
  line: 4
  column: 20
```

#### 4. Duplicate Key Validation

Object keys must be unique within their scope.

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

---

## Escaping

### Escape Sequences

| Sequence | Result | Use Case |
|----------|--------|----------|
| `\\` | Backslash | Literal backslash |
| `\n` | Newline (LF) | Line breaks |
| `\r` | Carriage return (CR) | Windows line endings |
| `\t` | Tab | Indentation |
| `\(` | Left paren | Literal paren in value |
| `\)` | Right paren | Literal paren in value |

**Examples:**
```gbln
path<s128>(C:\\Users\\Alice\\Documents)
multiline<s256>(Line 1\nLine 2\nLine 3)
formatted<s64>(Name:\tAlice\tAge:\t25)
parens<s32>(Formula: \(x + 1\) * \(x - 1\))
```

### Special Cases

#### Angle Brackets in Values

**No escaping needed** - angle brackets `<>` are automatically handled as value content:

```gbln
html<s256>(<h1>Hello</h1>)
xml<s512>(<user><name>Alice</name></user>)
generic<s64>(Vec<String>)
comparison<s32>(x < 10 && y > 5)
template<s128>(<div class="container">Content</div>)
```

**How it works:**
- Parser reads type hint first: `<s256>`
- Then reads value until matching closing paren: `(<h1>Hello</h1>)`
- All `<>` inside the parens are part of the value

#### Nested Parentheses in Values

**Automatically handled** through depth tracking:

```gbln
formula<s64>(f(x) = (x + 1) * (x - 1))
nested<s128>(outer(inner(deepest(x))))
lisp<s256>((lambda (x) (* x x)) 5)
```

**How it works:**
- Parser tracks parenthesis depth
- Only outermost closing `)` ends the value
- Inner parens are part of the value content

---

## Grammar (EBNF)

```ebnf
(* GBLN Grammar v1.1 *)

document     = value ;
value        = object | array | primitive ;

(* Objects *)
object       = "{" , [ key_values ] , "}" ;
key_values   = key_value , { whitespace , key_value } ;

(* Key-Value Pairs *)
key_value    = key , "<" , type_hint , ">" , "(" , raw_value , ")" ;
key          = identifier ;

(* Arrays *)
array        = "[" , [ array_items ] , "]" ;
array_items  = typed_array | mixed_array | object_array ;

(* Homogeneous array: all items same type *)
typed_array  = key , "<" , type_hint , ">" , "[" , [ values ] , "]" ;
values       = raw_value , { whitespace , raw_value } ;

(* Mixed array: each item explicitly typed *)
mixed_array  = typed_value , { whitespace , typed_value } ;
typed_value  = "<" , type_hint , ">" , "(" , raw_value , ")" ;

(* Object array *)
object_array = object , { whitespace , object } ;

(* Primitives *)
primitive    = key_value ;

(* Values *)
raw_value    = { char - ")" | escaped_char } ;
escaped_char = "\\" , ( "\\" | "(" | ")" | "n" | "r" | "t" ) ;

(* Type Hints *)
type_hint    = type_base , [ size ] ;
type_base    = "s" | "i" | "u" | "f" | "b" | "n" ;
size         = digit , { digit } ;

(* Basic Elements *)
identifier   = ( letter | "_" ) , { letter | digit | "_" } ;
digit        = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" ;
letter       = "a".."z" | "A".."Z" ;
whitespace   = " " | "\t" | "\n" | "\r" ;

(* Comments *)
comment      = ":|" , { char - "\n" } , "\n" ;

(* Note: Comments are preprocessed and stripped before parsing *)
```

---

## Error Handling

### Error Categories

1. **Syntax Errors**: Invalid GBLN syntax
2. **Type Errors**: Type mismatch or invalid type
3. **Range Errors**: Value out of range for type
4. **Validation Errors**: Failed validation rules
5. **Structure Errors**: Invalid structure (duplicate keys, etc.)

### Error Message Format

```
Error: <error_type>
  at field: <field_path>
  <error_details>
  line: <line_number>
  column: <column_number>
  
  suggestion: <helpful_suggestion>
```

### Example Error Messages

```
Error: Integer out of range
  at field: user.age
  value: 999
  type: i8
  valid range: -128 to 127
  line: 5
  column: 15
  
  suggestion: Use i16 or i32 for larger values

---

Error: String exceeds maximum length
  at field: user.name
  value: "VeryLongNameThatExceedsTheLimit"
  actual: 33 characters
  maximum: 32 characters (s32)
  line: 3
  column: 10
  
  suggestion: Use s64 for longer strings or shorten the value

---

Error: Type validation failed
  at field: settings.port
  expected: unsigned integer (u16)
  received: "abc" (not parseable as integer)
  line: 8
  column: 25
  
  suggestion: Ensure the value is a valid unsigned integer

---

Error: Duplicate key in object
  key: "id"
  first occurrence: line 2, column 5
  duplicate: line 4, column 5
  
  suggestion: Remove duplicate key or rename one of them

---

Error: Unexpected token
  expected: ')'
  found: ']'
  line: 10
  column: 30
  
  suggestion: Check for matching parentheses
```

---

## Implementation Requirements

### Must Support
- ✅ All integer types (i8-i64, u8-u64)
- ✅ All float types (f32, f64)
- ✅ All string types (s2-s1024)
- ✅ Boolean and null types
- ✅ Objects with nested structures
- ✅ Homogeneous arrays
- ✅ Mixed arrays
- ✅ Array of objects
- ✅ Comments (:|)
- ✅ All escape sequences
- ✅ Parse-time validation
- ✅ Detailed error messages

### Should Support
- ✅ UTF-8 encoding
- ✅ Line and column tracking
- ✅ Streaming parsing (for large files)
- ✅ Pretty printing
- ✅ Round-trip parsing (parse → serialize → parse)

### Optional Features
- ⚠️ Schema validation (external schema file)
- ⚠️ Custom type extensions
- ⚠️ Binary serialization format
- ⚠️ Compression

---

## Version History

### v1.1 (2025-01-20)
- Added s2, s4, s512, s1024 string types
- Clarified UTF-8 character counting
- Added duplicate key validation
- Improved error message format
- Added nested parentheses handling details

### v1.0 (2025-01-15)
- Initial specification release
- Core syntax defined
- Type system established
- Validation rules specified

---

*GBLN Specification v1.1*  
*© 2025 Vivian Voss - CC0 Public Domain*
