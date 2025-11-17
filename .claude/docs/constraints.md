# MetaObjects Constraint System

**Provider-based constraint registration for metadata validation**

**Status:** ✅ Fully Integrated into MetaDataRegistry (v6.2.0+)

---

## Overview

The MetaObjects constraint system validates metadata structure and properties during construction and modification.

**Key Principle:** Provider-based registration with runtime enforcement.

---

## Constraint Types

### PlacementConstraint

**Purpose:** Control where metadata can be placed in the hierarchy

**Example:** "Field can only be child of Object"

```java
public class FieldPlacementConstraint implements PlacementConstraint {
    @Override
    public boolean appliesTo(MetaData parent, MetaData child) {
        return child instanceof MetaField;
    }

    @Override
    public void validate(MetaData parent, MetaData child, ValidationContext context) {
        if (!(parent instanceof MetaObject)) {
            throw new ConstraintViolationException(
                "Field '" + child.getName() + "' can only be placed under MetaObject");
        }
    }
}
```

### ValidationConstraint

**Purpose:** Validate metadata properties and attributes

**Example:** "Name must match pattern"

```java
public class NamePatternConstraint implements ValidationConstraint {
    private final Pattern pattern = Pattern.compile("^[a-zA-Z][a-zA-Z0-9_]*$");

    @Override
    public boolean appliesTo(MetaData metadata) {
        return true; // Applies to all metadata
    }

    @Override
    public void validate(MetaData metadata, String propertyName, ValidationContext context) {
        if ("name".equals(propertyName)) {
            if (!pattern.matcher(metadata.getName()).matches()) {
                throw new ConstraintViolationException(
                    "Name must start with letter and contain only letters, numbers, underscores");
            }
        }
    }
}
```

---

## Constraint Registration

### Provider Pattern

**All constraints registered via MetaDataTypeProvider:**

```java
public class CoreTypesMetaDataProvider implements MetaDataTypeProvider {

    @Override
    public void registerTypes(MetaDataRegistry registry) {
        // Register types
        MetaObject.registerTypes(registry);
        MetaField.registerTypes(registry);

        // Register constraints
        registerConstraints(registry);
    }

    private void registerConstraints(MetaDataRegistry registry) {
        // Placement constraints
        registry.addPlacementConstraint(new FieldPlacementConstraint());
        registry.addPlacementConstraint(new AttributePlacementConstraint());

        // Validation constraints
        registry.addValidationConstraint(new NamePatternConstraint());
        registry.addValidationConstraint(new RequiredAttributeConstraint());
    }

    @Override
    public int getPriority() {
        return 10; // Core constraints have high priority
    }
}
```

---

## Constraint Naming Convention

**Format:** `{domain}:{type}:{operation}`

**Examples:**
- `metadata:field:placement` - Field placement constraint
- `metadata:object:validation` - Object validation constraint
- `enterprise:security:validation` - Enterprise security validation

**Why:** Clear identification, prevents conflicts, enables filtering

---

## Unified Registry Architecture

### Before (Separate Registries):
```java
// OLD - Separate constraint registry
ConstraintRegistry constraintRegistry = ...;
MetaDataRegistry typeRegistry = ...;

// Two places to look for configuration
```

### After (Integrated):
```java
// NEW - Single unified registry
MetaDataRegistry registry = ...;

// Types and constraints in one place
registry.registerType(MetaField.class, def -> ...);
registry.addValidationConstraint(new NamePatternConstraint());
```

**Benefits:**
- ✅ Single source of truth
- ✅ Unified API
- ✅ Simpler debugging
- ✅ Better performance

---

## Enforcement

### Automatic Enforcement

**Constraints enforced automatically during:**

1. **addChild()** - When adding child metadata
   ```java
   metaObject.addChild(metaField); // Checks placement constraints
   ```

2. **setAttribute()** - When setting attributes
   ```java
   metaData.setAttribute("name", "value"); // Validates attribute
   ```

3. **Construction** - During metadata creation
   ```java
   new MetaField("fieldName"); // Validates name pattern
   ```

### Enforcement Workflow

```java
public void enforceConstraintsOnAddChild(MetaData parent, MetaData child) {
    ValidationContext context = ValidationContext.forAddChild(parent, child);

    // Get all constraints from unified registry
    List<Constraint> allConstraints = metaDataRegistry.getAllValidationConstraints();

    // 1. Check placement constraints
    for (Constraint constraint : allConstraints) {
        if (constraint instanceof PlacementConstraint) {
            PlacementConstraint pc = (PlacementConstraint) constraint;
            if (pc.appliesTo(parent, child)) {
                pc.validate(parent, child, context);
            }
        }
    }

    // 2. Validate child properties
    for (Constraint constraint : allConstraints) {
        if (constraint instanceof ValidationConstraint) {
            ValidationConstraint vc = (ValidationConstraint) constraint;
            if (vc.appliesTo(child)) {
                vc.validate(child, child.getName(), context);
            }
        }
    }
}
```

---

## Built-In Constraints

### Core Placement Constraints

**FieldPlacementConstraint:**
- Fields can only be children of Objects
- Prevents field hierarchies

**AttributePlacementConstraint:**
- Attributes can be children of any MetaData
- Flexible attribute system

**ViewPlacementConstraint:**
- Views can only be children of Objects
- View organization

### Core Validation Constraints

**NamePatternConstraint:**
- Names must match: `^[a-zA-Z][a-zA-Z0-9_]*$`
- Starts with letter
- Contains only letters, numbers, underscores

**RequiredAttributeConstraint:**
- Certain metadata types require specific attributes
- Example: DecimalField requires @precision

**TypeSubTypeConstraint:**
- Validates type/subType combinations
- Prevents invalid type hierarchies

---

## Custom Constraints

### Creating Custom Constraint

```java
public class EmailValidationConstraint implements ValidationConstraint {
    private static final Pattern EMAIL_PATTERN =
        Pattern.compile("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$");

    @Override
    public boolean appliesTo(MetaData metadata) {
        // Only apply to fields with subType "email"
        return metadata instanceof MetaField &&
               "email".equals(metadata.getSubTypeName());
    }

    @Override
    public void validate(MetaData metadata, String propertyName, ValidationContext context) {
        if ("defaultValue".equals(propertyName)) {
            String value = metadata.getAttributeValue("defaultValue");
            if (value != null && !EMAIL_PATTERN.matcher(value).matches()) {
                throw new ConstraintViolationException(
                    "Default value must be valid email address");
            }
        }
    }

    @Override
    public String getName() {
        return "custom:email:validation";
    }
}
```

### Registering Custom Constraint

```java
public class CustomTypesMetaDataProvider implements MetaDataTypeProvider {

    @Override
    public void registerTypes(MetaDataRegistry registry) {
        // Register custom types
        EmailField.registerTypes(registry);

        // Register custom constraints
        registry.addValidationConstraint(new EmailValidationConstraint());
    }

    @Override
    public int getPriority() {
        return 50; // After core (10), before application (100)
    }
}
```

### ServiceLoader Registration

```
# META-INF/services/com.draagon.meta.loader.types.MetaDataTypeProvider
com.example.custom.CustomTypesMetaDataProvider
```

---

## Constraint Priority

**Provider priority determines constraint order:**

```java
Priority 10  - Core constraints (framework)
Priority 25  - Enterprise constraints
Priority 50  - Plugin constraints
Priority 100 - Application constraints
```

**Lower priority = executed first**

---

## Performance

### Optimizations

**Single Enforcement Path:**
- Before: 4 separate constraint checks
- After: 1 unified loop
- Result: **3x fewer constraint calls**

**Efficient Storage:**
- Constraints cached in ArrayList
- O(n) iteration only when needed
- No duplicate processing

**Lazy Evaluation:**
- `appliesTo()` filters before validation
- Only relevant constraints executed
- Minimal overhead

---

## Testing Constraints

### Unit Test Example

```java
@Test
public void testFieldPlacementConstraint() {
    MetaObject obj = new MetaObject("TestObject");
    MetaField field = new IntegerField("testField");

    // Should succeed
    obj.addChild(field);

    // Should fail - field under field not allowed
    try {
        field.addChild(new StringField("nested"));
        fail("Should throw ConstraintViolationException");
    } catch (ConstraintViolationException e) {
        // Expected
    }
}
```

### Running Constraint Tests

```bash
cd ~/Development/metaobjects-core/metadata
mvn test -Dtest=ConstraintSystemTest
```

---

## Debugging Constraints

### Enable Constraint Logging

```java
// In logback.xml or logback-test.xml
<logger name="com.draagon.meta.constraint" level="DEBUG"/>
```

### Validation Context

```java
ValidationContext context = ValidationContext.forAddChild(parent, child);
context.setDebugMode(true); // Enables detailed logging

// Shows which constraints applied
// Shows validation steps
// Shows failures with context
```

---

## Best Practices

### DO:

✅ **Register via Provider** - Use MetaDataTypeProvider
✅ **Name Consistently** - Follow `domain:type:operation` pattern
✅ **Filter Efficiently** - `appliesTo()` should be fast
✅ **Fail Fast** - Throw clear exceptions early
✅ **Document Constraints** - Explain what and why

### DON'T:

❌ **Add Directly to Registry** - Use provider pattern
❌ **Skip Naming Convention** - Maintain consistency
❌ **Validate in appliesTo()** - Use separate validate() method
❌ **Catch and Ignore** - Let exceptions propagate
❌ **Create Rigid Constraints** - Allow extensibility

---

## Common Patterns

### Conditional Validation

```java
@Override
public void validate(MetaData metadata, String propertyName, ValidationContext context) {
    if (metadata.hasAttribute("@required")) {
        boolean required = metadata.getBooleanAttribute("@required", false);
        if (required && metadata.getAttributeValue(propertyName) == null) {
            throw new ConstraintViolationException(
                propertyName + " is required");
        }
    }
}
```

### Cross-Field Validation

```java
@Override
public void validate(MetaData metadata, String propertyName, ValidationContext context) {
    if (metadata instanceof DecimalField) {
        int precision = metadata.getIntAttribute("@precision", 19);
        int scale = metadata.getIntAttribute("@scale", 2);

        if (scale > precision) {
            throw new ConstraintViolationException(
                "Scale cannot exceed precision");
        }
    }
}
```

### Hierarchical Constraints

```java
@Override
public boolean appliesTo(MetaData parent, MetaData child) {
    // Only apply if parent is specific type
    return parent instanceof MetaObject &&
           "entity".equals(parent.getSubTypeName());
}
```

---

## Migration from Old System

### JSON Constraints (Deprecated)

**Old Way:**
```json
{
  "constraint": {
    "name": "field:placement",
    "type": "placement",
    "target": {"type": "field"}
  }
}
```

**New Way:**
```java
registry.addPlacementConstraint(new FieldPlacementConstraint());
```

**Benefits:**
- Compile-time safety
- Better IDE support
- Easier debugging
- Performance improvements

---

## See Also

- [ARCHITECTURE.md](../ARCHITECTURE.md) - Provider-based registration
- [type-system.md](type-system.md) - Field types and validation
- [testing.md](testing.md) - Testing constraints
- [KNOWN-ISSUES.md](../KNOWN-ISSUES.md) - Common constraint problems

---

**Last Updated:** 2025-11-15
**Version:** 6.2.0+
