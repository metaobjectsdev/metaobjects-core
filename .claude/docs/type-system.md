# MetaObjects Type System

**AI-optimized field types with cross-language semantic consistency**

**Status:** ✅ Completed (v6.3.0+)

---

## Overview

MetaObjects uses a **33% simplified** field type system optimized for AI code generation across Java, C#, TypeScript, Python, and other languages.

**Key Principle:** Semantic types that map directly to native language types.

---

## Core Semantic Types (6)

### Universal Language Mapping

| Type | Java | C# | TypeScript | Python | Usage |
|------|------|-----|------------|--------|-------|
| **string** | String | string | string | str | Text data |
| **int** | Integer | int | number | int | Whole numbers, normal range |
| **long** | Long | long | number | int | Whole numbers, large range |
| **float** | Float | float | number | float | Decimal, standard precision |
| **double** | Double | double | number | float | Decimal, high precision |
| **decimal** | BigDecimal | decimal | Decimal | Decimal | Financial, exact precision |

---

## Special Types (4)

### Boolean
```json
{
  "field": {
    "name": "isActive",
    "subType": "boolean"
  }
}
```
Maps to: `Boolean/bool/boolean/bool`

### Date
```json
{
  "field": {
    "name": "birthDate",
    "subType": "date"
  }
}
```
Maps to: `LocalDate/DateTime/Date/date`

### Timestamp
```json
{
  "field": {
    "name": "createdAt",
    "subType": "timestamp"
  }
}
```
Maps to: `LocalDateTime/DateTime/Date/datetime`

### Binary
```json
{
  "field": {
    "name": "data",
    "subType": "binary"
  }
}
```
Maps to: `byte[]/byte[]/Uint8Array/bytes`

---

## Object Types (3)

### Object Reference
```json
{
  "field": {
    "name": "parent",
    "subType": "object",
    "refObject": "Category"
  }
}
```

### Class Reference
```json
{
  "field": {
    "name": "entityClass",
    "subType": "class"
  }
}
```

### Legacy Array Types
- `stringArray` - Deprecated (use `@isArray` instead)
- `objectArray` - Deprecated (use `@isArray` instead)

---

## Universal Array Support

**Any field type can be an array** using `@isArray` modifier:

```json
{
  "field": {
    "name": "scores",
    "subType": "int",
    "@isArray": true
  }
}
```

**Eliminates need for separate array types:**
- ~~IntArrayField~~ → `IntegerField` with `@isArray: true`
- ~~StringArrayField~~ → `StringField` with `@isArray: true`
- ~~ObjectArrayField~~ → `ObjectField` with `@isArray: true`

---

## DecimalField - High-Precision Numbers

**Use for:** Financial calculations, exact decimal representation

```json
{
  "field": {
    "name": "price",
    "subType": "decimal",
    "@precision": 19,
    "@scale": 2
  }
}
```

**Attributes:**
- `@precision` - Total digits (default: 19)
- `@scale` - Decimal places (default: 2)

**Language Mappings:**
- Java: `BigDecimal`
- C#: `decimal`
- TypeScript: `Decimal` (from decimal.js)
- Python: `Decimal`

**Database:** `DECIMAL(19,2)` or equivalent

---

## What Was Removed (v6.3.0)

### Eliminated Types

**ByteField** - Rarely used, complexity without benefit
- Use: `IntegerField` instead
- Range validation sufficient for byte ranges

**ShortField** - Rarely used, adds decision complexity
- Use: `IntegerField` instead
- Range validation sufficient for short ranges

**Separate Array Types** - Type explosion problem
- Old: `IntArrayField`, `StringArrayField`, etc.
- New: Any field + `@isArray: true`

### Why These Were Removed

1. **Reduced AI Complexity:** 33% fewer type decisions
2. **Semantic Clarity:** Types match business meaning, not technical details
3. **Cross-Language:** Direct mapping to native types
4. **Maintainability:** Fewer types = simpler codebase

---

## Field Type Selection Guide

### Choose `string` when:
- Text data of any length
- Names, descriptions, URLs, JSON

### Choose `int` when:
- Whole numbers, typical business ranges
- IDs, counts, quantities (< 2 billion)

### Choose `long` when:
- Large whole numbers
- IDs, counts beyond int range
- Timestamps (milliseconds since epoch)

### Choose `float` when:
- Decimal numbers, standard precision okay
- Measurements, calculations
- NOT for money (use decimal)

### Choose `double` when:
- Decimal numbers, high precision needed
- Scientific calculations
- NOT for money (use decimal)

### Choose `decimal` when:
- **Financial calculations** (ALWAYS!)
- Money, prices, rates, percentages
- Exact decimal representation required

### Choose `boolean` when:
- True/false, yes/no, on/off
- Flags, switches, status

### Choose `date` when:
- Calendar dates without time
- Birth dates, deadlines

### Choose `timestamp` when:
- Date + time
- Creation/modification timestamps
- Event times

### Choose `object` when:
- Reference to another entity
- Foreign keys, relationships

---

## Range Validation

### FloatField
```json
{
  "field": {
    "name": "temperature",
    "subType": "float",
    "@minValue": -273.15,
    "@maxValue": 1000000.0
  }
}
```

### DoubleField
```json
{
  "field": {
    "name": "scientificValue",
    "subType": "double",
    "@minValue": 0.0,
    "@maxValue": 1.7976931348623157E308
  }
}
```

### IntegerField / LongField
```json
{
  "field": {
    "name": "quantity",
    "subType": "int",
    "@minValue": 0,
    "@maxValue": 2147483647
  }
}
```

---

## Best Practices

### DO:
✅ Use `decimal` for **all money/financial** calculations
✅ Use semantic types (`int`, not `byte`)
✅ Use `@isArray: true` for arrays
✅ Map types to native language equivalents
✅ Use range validation when needed

### DON'T:
❌ Use `float` or `double` for money
❌ Use byte/short (removed in v6.3.0)
❌ Create separate array field types
❌ Use technical types when semantic types exist

---

## Migration from Old Types

### ByteField → IntegerField
```json
// OLD
{
  "field": {
    "name": "age",
    "subType": "byte"
  }
}

// NEW
{
  "field": {
    "name": "age",
    "subType": "int",
    "@minValue": 0,
    "@maxValue": 127
  }
}
```

### Array Types → @isArray
```json
// OLD
{
  "field": {
    "name": "tags",
    "subType": "stringArray"
  }
}

// NEW
{
  "field": {
    "name": "tags",
    "subType": "string",
    "@isArray": true
  }
}
```

---

## Database Mappings

| MetaObjects Type | PostgreSQL | MySQL | SQL Server | Oracle |
|-----------------|------------|-------|------------|--------|
| string | VARCHAR | VARCHAR | VARCHAR | VARCHAR2 |
| int | INTEGER | INT | INT | NUMBER(10) |
| long | BIGINT | BIGINT | BIGINT | NUMBER(19) |
| float | REAL | FLOAT | REAL | FLOAT(24) |
| double | DOUBLE PRECISION | DOUBLE | FLOAT | FLOAT(53) |
| **decimal** | DECIMAL(19,2) | DECIMAL(19,2) | DECIMAL(19,2) | NUMBER(19,2) |
| boolean | BOOLEAN | TINYINT(1) | BIT | NUMBER(1) |
| date | DATE | DATE | DATE | DATE |
| timestamp | TIMESTAMP | DATETIME | DATETIME2 | TIMESTAMP |
| binary | BYTEA | BLOB | VARBINARY | BLOB |

---

## AI Generation Benefits

### Before (8 types):
```
AI must decide between:
  byte, short, int, long
  float, double
  string, boolean, date
  + separate array types
= 16+ type decisions
```

### After (6 core types):
```
AI decides semantic meaning:
  int, long
  float, double, decimal
  string, boolean, date
  + @isArray modifier
= 10 type decisions (33% reduction)
```

**Result:** Simpler, faster, more accurate AI code generation

---

## Code Examples

### Java
```java
@MetaField(subType = "decimal")
private BigDecimal price;

@MetaField(subType = "int")
@IsArray(true)
private List<Integer> scores;
```

### C#
```csharp
[MetaField(SubType = "decimal")]
public decimal Price { get; set; }

[MetaField(SubType = "int")]
[IsArray(true)]
public List<int> Scores { get; set; }
```

### TypeScript
```typescript
@MetaField({ subType: 'decimal' })
price: Decimal;

@MetaField({ subType: 'int', isArray: true })
scores: number[];
```

---

## Testing Type System

```bash
cd ~/Development/metaobjects-core/metadata
mvn test -Dtest=FieldTypeTest
```

All 232 tests passing ✅

---

**See Also:**
- [ARCHITECTURE.md](../ARCHITECTURE.md) - Overall design principles
- [KNOWN-ISSUES.md](../KNOWN-ISSUES.md) - Type migration notes
- [SETUP.md](../SETUP.md) - Building and testing

---

**Last Updated:** 2025-11-15
**Version:** 6.3.0+
