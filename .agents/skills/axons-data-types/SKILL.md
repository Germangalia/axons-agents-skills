name: axons-data-types
description: AXONS data type system including primitives, string types (CiString, UUID v7), binary (Base64URL), temporal types (ISO 8601), enumerations, and validation rules. Use when defining message schemas, validating data, or implementing serialization.

# AXONS Data Types

## When to Load

Activate this skill when the task involves:
- Defining or validating message schema fields
- Working with UUID v7 identifiers
- Implementing string format constraints (CiString, Base64URL)
- Handling temporal types (dateTime, date, time)
- Defining or extending enumeration types
- Serialization/deserialization of AXONS data

## Key References

- `content/docs/axons-specification/01-architecture/01-04-data-types.md`

## Primitive Types

| Type | JSON Type | Description | Example |
|------|-----------|-------------|---------|
| `string` | string | Text data | `"Hello"` |
| `integer` | number | 32-bit signed whole number | `42`, `-100` |
| `decimal` | number | Double precision float | `3.14159` |
| `boolean` | boolean | True/false | `true`, `false` |
| `null` | null | Explicit absence | `null` |
| `array` | array | Ordered collection (same type) | `["a", "b"]` |
| `object` | object | Key-value structure | `{"key": "value"}` |
| `dateTime` | string | ISO 8601 UTC with ms | `"2026-03-15T10:00:00.000Z"` |
| `UUID` | string | RFC 9562 UUID v7 | `"0191a7e0-5c30-7f60-b234-8c9e4abc1234"` |
| `binary` | string | Base64URL (RFC 4648) | `"SGVsbG8"` |

## Integer Ranges

| Range | Min | Max |
|-------|-----|-----|
| `byte` | -128 | 127 |
| `unsignedByte` | 0 | 255 |
| `short` | -32,768 | 32,767 |
| `unsignedShort` | 0 | 65,535 |
| `int` | -2,147,483,648 | 2,147,483,647 |
| `unsignedInt` | 0 | 4,294,967,295 |

## Decimal Precision

| Use Case | Precision | Example |
|----------|-----------|---------|
| Percentage | 2 decimal places | `99.99` |
| Temperature | 1 decimal place | `23.5` |
| Coordinates | 6 decimal places | `41.402778` |

## String Types: CiString (Case-Insensitive)

| Type | Max Length | Use Case |
|------|------------|----------|
| CiString20 | 20 | Identifiers, short codes |
| CiString50 | 50 | Model names, serial numbers |
| CiString255 | 255 | URLs, longer identifiers |
| CiString500 | 500 | Extended data fields |

## Format Constraints

| Format | Pattern | Example |
|--------|---------|---------|
| `alpha` | `[a-zA-Z]+` | `"Hello"` |
| `alphanumeric` | `[a-zA-Z0-9]+` | `"Device001"` |
| `email` | RFC 5322 | `"user@example.com"` |
| `uri` | RFC 3986 | `"https://example.com/path"` |
| `uuid` | RFC 9562 v7 | `"0191a7e0-5c30-7f60-b234-8c9e4abc1234"` |

## UUID v7 Format

UUID v7 is the MANDATORY identifier format for all AXONS entities and messages. It is time-sortable.

**Pattern**: `^[0-9a-f]{8}-[0-9a-f]{4}-7[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$`

Key property: the version digit (position 13) is always `7`, and the variant digits (position 17) are in `[89ab]`.

## Binary (Base64URL)

- Uses Base64URL alphabet: `A-Z`, `a-z`, `0-9`, `-`, `_`
- Padding characters (`=`) MUST NOT be included
- No line breaks or whitespace

## Temporal Types

| Format | Pattern | Requirements |
|--------|---------|-------------|
| `dateTime` | `YYYY-MM-DDTHH:mm:ss.sssZ` | UTC, milliseconds required |
| `date` | `YYYY-MM-DD` | No timezone |
| `time` | `HH:mm:ssZ` | UTC suffix required |

### DateTime Rules
- MUST be UTC (`Z` suffix)
- MUST include milliseconds (3 digits)
- MUST NOT include timezone offset (`+HH:MM` or `-HH:MM`)

## Enumeration Types

- Values MUST be PascalCase (or lowercase consistently within type)
- Case-sensitive matching
- Unknown values MUST be rejected
- Naming pattern: `{Name}EnumType`
- New values only through protocol version update

## Null Semantics

- `null` ≠ field omission (both are valid for optional fields)
- Required fields (cardinality 1 or 1..1) MUST NOT be null
- Implementations MUST treat `null` and omission as equivalent for optional fields

## Array Rules

- All elements same type
- Empty arrays valid unless min cardinality specified
- Insertion order preserved
- Duplicates allowed unless prohibited
- Nested arrays NOT permitted
- Nesting depth SHOULD NOT exceed 5 levels

## Universal Validation Rules

1. Numeric values use appropriate JSON type (integer vs float)
2. dateTime values in ISO 8601 UTC with Z suffix
3. Strings MUST be valid UTF-8
4. UUID v7 identifiers follow RFC 9562 format
5. Enum values match exactly (case-sensitive)
6. Unknown enum values rejected
7. Required fields present and non-null
8. Optional fields may be omitted or null

## Common Pitfalls

- Using `datetime` without milliseconds → invalid format
- Forgetting `Z` suffix on UTC timestamps
- Including `=` padding in Base64URL encoded values
- Using UUID v4 instead of UUID v7 → not time-sortable
- Using JWT format for opaque tokens → exposes metadata
- Nested arrays in message payload → invalid
- Case mismatch in enum values → rejected
- Overriding the `action` naming convention (must be PascalCase)
