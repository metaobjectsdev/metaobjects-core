# MetaObjects Testing Guide

**Unit test patterns and best practices**

---

## Overview

MetaObjects uses JUnit 4 for testing with special patterns to handle ServiceLoader and shared registry concerns.

**Key Principle:** Shared registry for consistent state across tests.

---

## Test Base Classes

### SharedRegistryTestBase (Use This 95% of the Time)

**Purpose:** Standard tests that need consistent MetaDataRegistry state

```java
public class YourNewTest extends SharedRegistryTestBase {

    @Test
    public void testSomething() {
        // Shared registry already initialized
        MetaDataRegistry registry = getSharedRegistry();

        // Test your code
        MetaObject obj = registry.getMetaObjectByName("User");
        assertNotNull(obj);
    }
}
```

**Benefits:**
- Shared registry across all tests
- No ServiceLoader conflicts
- Fast test execution
- Consistent state

**When to use:**
- Standard unit tests
- Constraint tests
- Type registration tests
- Most metadata tests

---

### IsolatedRegistryTestBase (Rare, Special Cases Only)

**Purpose:** Tests that need completely isolated registry state

```java
public class SpecialIsolatedTest extends IsolatedRegistryTestBase {

    @Override
    protected void setupRegistry() {
        // Custom registry setup
        registry = createFreshRegistry();
        // Register only specific types needed
    }

    @Test
    public void testWithIsolation() {
        // Completely isolated registry
        // No other test interference
    }
}
```

**When to use:**
- Testing provider registration
- Testing constraint conflicts
- Testing registry edge cases
- <5% of tests

---

## Test Patterns

### Testing Metadata Creation

```java
@Test
public void testMetaObjectCreation() {
    MetaObject obj = new MetaObject("TestObject");
    assertEquals("TestObject", obj.getName());
    assertEquals("object", obj.getTypeName());
}
```

### Testing Fields

```java
@Test
public void testFieldTypes() {
    MetaObject obj = new MetaObject("User");

    IntegerField idField = new IntegerField("id");
    StringField nameField = new StringField("name");

    obj.addChild(idField);
    obj.addChild(nameField);

    assertEquals(2, obj.getMetaFields().size());
}
```

### Testing Constraints

```java
@Test
public void testPlacementConstraints() {
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

### Testing Providers

```java
@Test
public void testProviderRegistration() {
    MetaDataRegistry registry = getSharedRegistry();

    // Verify types registered
    assertNotNull(registry.getTypeDefinition("field", "int"));
    assertNotNull(registry.getTypeDefinition("field", "string"));
}
```

---

## Database Testing

### In-Memory Derby

```java
@Test
public void testDatabasePersistence() throws Exception {
    // 1. Setup in-memory database
    EmbeddedDataSource dataSource = new EmbeddedDataSource();
    dataSource.setDatabaseName("memory:testdb");
    dataSource.setCreateDatabase("create");

    // 2. Initialize ObjectManagerDB
    ObjectManagerDB objectManager = new ObjectManagerDB();
    objectManager.setDriverClass("com.metaobjects.manager.db.driver.DerbyDriver");
    objectManager.setDataSource(dataSource);

    // 3. Test persistence
    ObjectConnection conn = objectManager.getConnection();
    try {
        // Create, read, update, delete
    } finally {
        conn.close();
    }
}
```

---

## Running Tests

### All Tests

```bash
cd ~/Development/metaobjects-core
mvn test
```

### Specific Module

```bash
cd metadata
mvn test
```

### Specific Test Class

```bash
mvn test -Dtest=ConstraintSystemTest
```

### Single Test Method

```bash
mvn test -Dtest=ConstraintSystemTest#testPlacementConstraints
```

### With Debug Logging

```bash
mvn test -X
```

---

## Test Organization

### Standard Test Structure

```
src/test/java/
└── com/draagon/meta/
    ├── MetaObjectTest.java
    ├── MetaFieldTest.java
    ├── constraint/
    │   ├── ConstraintSystemTest.java
    │   └── PlacementConstraintTest.java
    ├── loader/
    │   ├── SimpleLoaderTest.java
    │   └── FileLoaderTest.java
    └── manager/
        └── ObjectManagerTest.java
```

### Test Resources

```
src/test/resources/
├── metadata/
│   ├── test-metadata.json
│   └── test-overlay.json
├── logback-test.xml
└── META-INF/
    └── services/
        └── com.draagon.meta.loader.types.MetaDataTypeProvider
```

---

## Best Practices

### DO:

✅ **Extend SharedRegistryTestBase** for most tests
✅ **Use @Test annotation** from JUnit 4
✅ **Clean up resources** in @After or try-finally
✅ **Test one thing** per test method
✅ **Use descriptive test names** (testFieldPlacementConstraint)
✅ **Assert expectations** clearly

### DON'T:

❌ **Create separate registries** unless absolutely needed
❌ **Modify shared state** without cleanup
❌ **Use Thread.sleep()** in tests
❌ **Test multiple things** in one method
❌ **Leave resources open** (connections, files)

---

## Common Test Issues

### ServiceLoader Conflicts

**Problem:** Tests fail with duplicate constraint errors on GitHub Actions

**Solution:** Use `SharedRegistryTestBase`

### Registry State

**Problem:** Tests pass individually but fail together

**Solution:**
```java
@After
public void tearDown() {
    // Clear any state modified by test
    MetaDataCache.clear();
}
```

### Constraint Naming

**Problem:** Constraint tests fail with "Name required"

**Solution:** Follow naming convention: `domain:type:operation`

```java
@Override
public String getName() {
    return "metadata:field:placement";
}
```

---

## Test Coverage

### Current Status

**Total Tests:** 232 passing ✅

**Coverage by Module:**
- metadata: 150+ tests
- core: 40+ tests
- codegen: 15+ tests
- objectmanager: 10+ tests
- web: 15+ tests

### Running with Coverage

```bash
mvn clean test jacoco:report
# Report in: target/site/jacoco/index.html
```

---

## Mock Objects

### Mocking MetaData

```java
@Test
public void testWithMockMetaData() {
    MetaObject mockObject = new MetaObject("MockObject");
    mockObject.addChild(new IntegerField("id"));
    mockObject.addChild(new StringField("name"));

    // Test code that uses mockObject
}
```

### Mocking Database

```java
@Test
public void testWithMockDatabase() throws Exception {
    // Use in-memory Derby - no mocking needed!
    EmbeddedDataSource dataSource = new EmbeddedDataSource();
    dataSource.setDatabaseName("memory:testdb");
    dataSource.setCreateDatabase("create");

    // Full functional database in memory
}
```

---

## Debugging Tests

### Enable Debug Logging

**Edit src/test/resources/logback-test.xml:**

```xml
<configuration>
    <!-- Change WARN to DEBUG -->
    <logger name="com.draagon.meta" level="DEBUG"/>

    <root level="DEBUG">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

### Run Single Test with Debug

```bash
mvn test -Dtest=YourTest -X
```

### IntelliJ IDEA

1. Right-click test method
2. Select "Debug 'testMethod()'"
3. Set breakpoints as needed

---

## Continuous Integration

### GitHub Actions

Tests run automatically on every push:

```yaml
# .github/workflows/build.yml
- name: Run tests
  run: mvn clean test
```

**Passes on:** Linux, macOS, Windows

---

## See Also

- [SETUP.md](../SETUP.md) - Build and test commands
- [constraints.md](constraints.md) - Testing constraints
- [persistence.md](persistence.md) - Database testing
- [KNOWN-ISSUES.md](../KNOWN-ISSUES.md) - Common test issues

---

**Last Updated:** 2025-11-15
