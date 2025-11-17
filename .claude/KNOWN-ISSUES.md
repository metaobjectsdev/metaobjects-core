# MetaObjects Core - Known Issues & Lessons Learned

**Common problems, gotchas, and hard-learned lessons**

**Last Updated:** 2025-11-15

---

## 🚨 Critical Lessons Learned

### DON'T Treat MetaData as Mutable Domain Objects

**❌ WRONG:**
```java
MetaObject userMeta = loader.getMetaObjectByName("User");
userMeta.addMetaField(new StringField("dynamicField")); // ❌ Runtime mutation
```

**✅ RIGHT:**
```java
MetaObject userMeta = loader.getMetaObjectByName("User");
MetaField field = userMeta.getMetaField("email"); // ✅ Read-only access
```

**Why:** MetaObjects is like Java's ClassLoader - load once, read many times. Treating it like a mutable entity breaks the read-optimized architecture and causes thread-safety issues.

---

### DON'T Replace WeakHashMap with Strong References

**❌ WRONG:**
```java
private final Map<Object, Object> cache = new ConcurrentHashMap<>(); // ❌ Causes memory leaks
```

**✅ RIGHT:**
```java
private final Map<Object, Object> cache = Collections.synchronizedMap(new WeakHashMap<>()); // ✅
```

**Why:** In OSGi environments, bundles can be unloaded. Strong references prevent garbage collection of computed caches, causing memory leaks. WeakHashMap allows GC cleanup when bundles unload.

---

###DON'T Create New MetaDataLoader Instances Frequently

**❌ WRONG:**
```java
for (String source : sources) {
    MetaDataLoader loader = new SimpleLoader(source); // ❌ Expensive!
    loader.init();
}
```

**✅ RIGHT:**
```java
MetaDataLoader appLoader = new SimpleLoader("appMetadata");
appLoader.setSourceURIs(allSources);
appLoader.init(); // ✅ Load once, use forever
```

**Why:** MetaDataLoader is analogous to ClassLoader - expensive initialization, meant to be created once per application context.

---

### DON'T Synchronize Read Operations After Loading

**❌ WRONG:**
```java
public synchronized MetaField getMetaField(String name) { // ❌ Kills performance
    return fieldCache.get(name);
}
```

**✅ RIGHT:**
```java
public MetaField getMetaField(String name) { // ✅ Lock-free reads
    return fieldCache.get(name); // Immutable after loading
}
```

**Why:** After loading phase, metadata is immutable. Synchronization on reads creates unnecessary contention and destroys the performance benefits of read-optimization.

---

### DON'T Use Single Concrete Classes with String-Based Subtype Checking

**❌ WRONG:**
```java
public class MetaValidator extends MetaData {
    public boolean isRequired() {
        return "required".equals(getSubTypeName()); // ❌ String checking
    }
}
```

**✅ RIGHT:**
```java
public abstract class MetaValidator extends MetaData { /* base class */ }

public class RequiredValidator extends MetaValidator {
    // ✅ Type-specific implementation
    @Override
    public void validate(Object value) { /* ... */ }
}
```

**Why:** Abstract Base + Concrete Subtypes pattern provides type safety, better APIs, and cleaner code. See [ARCHITECTURE.md](ARCHITECTURE.md) for details.

---

## ⚠️ OSGi Compatibility Pitfalls

### Issue: Memory Leaks When Bundles Unload

**Problem:** Strong references to computed caches prevent GC when OSGi bundles unload

**Solution:**
- Use `WeakHashMap` for computed/derived caches
- Use `ConcurrentHashMap` only for permanent metadata references
- Use `WeakReference` for ClassLoader references

**Example:**
```java
// PERMANENT CACHE - Strong references
private final Map<String, Object> permanentCache = new ConcurrentHashMap<>();

// COMPUTED CACHE - Weak references
private final Map<Object, Object> computedCache = Collections.synchronizedMap(new WeakHashMap<>());
```

**Status:** ✅ Implemented correctly throughout framework

---

### Issue: ServiceLoader Not Finding Providers in OSGi

**Problem:** `META-INF/services/` files not being read correctly in OSGi bundles

**Solution:**
1. Ensure `META-INF/services/` directory is in the bundle
2. Use proper ClassLoader for ServiceLoader:
   ```java
   ServiceLoader.load(MetaDataTypeProvider.class, this.getClass().getClassLoader())
   ```
3. Verify OSGi manifest includes service files

**Status:** ✅ Fixed with proper ClassLoader usage

---

## 🐛 Common Build Issues

### Issue: "Cannot Find Symbol" Errors

**Cause:** Module dependencies not resolved in local Maven repository

**Solution:**
```bash
mvn clean install  # Install all modules to ~/.m2/repository
```

**Prevention:** Always `mvn install` after pulling latest code

---

### Issue: Tests Fail with ServiceLoader Errors

**Cause:** Missing or incorrect `META-INF/services/` provider files

**Solution:**
1. Check file exists: `src/main/resources/META-INF/services/com.draagon.meta.loader.types.MetaDataTypeProvider`
2. Verify provider class name is correct (fully qualified)
3. Ensure provider class is on classpath

**Example Provider File:**
```
# META-INF/services/com.draagon.meta.loader.types.MetaDataTypeProvider
com.draagon.meta.field.FieldTypesMetaDataProvider
com.draagon.meta.validator.ValidatorTypesMetaDataProvider
```

---

### Issue: OSGi Manifest Generation Fails

**Cause:** Module not configured as OSGi bundle

**Solution:**
Ensure `pom.xml` has:
```xml
<packaging>bundle</packaging>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.felix</groupId>
            <artifactId>maven-bundle-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

---

### Issue: Schema Generation Doesn't Run

**Cause:** MetaObjects Maven Plugin not finding metadata files

**Solution:**
1. Verify metadata files in `src/main/resources/metadata/`
2. Run from `core` module: `cd core && mvn metaobjects:generate@gen-schemas`
3. Check plugin configuration in `core/pom.xml`

---

## 🔧 Development Gotchas

### Gotcha: Adding Validation to Core Types

**Problem:** Adding rigid validation to base classes prevents extensibility

**DON'T:**
```java
public class MetaField extends MetaData {
    private static final Set<String> ALLOWED_SUBTYPES = Set.of("string", "int"); // ❌ Rigid!

    public MetaField(String subType) {
        if (!ALLOWED_SUBTYPES.contains(subType)) { // ❌ Prevents plugins!
            throw new IllegalArgumentException("Invalid subtype");
        }
    }
}
```

**DO:**
```java
// Use constraint system for validation
// Add constraints in META-INF/constraints/*.json
// Allows extensibility through provider system
```

**Why:** Hardcoded validation prevents downstream implementations from extending subtypes through the provider system.

---

### Gotcha: Forgetting to Flush Caches After Structural Changes

**Problem:** After adding/removing child metadata during loading, caches may be stale

**Solution:**
```java
public synchronized void addChild(MetaData child) {
    children.add(child);
    flushCaches(); // ⚠️ Don't forget this!
}
```

**Why:** Computed caches (like field lookups) may return stale data if not flushed after structural changes.

---

### Gotcha: Using Wrong ClassLoader for Resource Loading

**Problem:** Resources not found in OSGi environments

**Solution:**
```java
// ❌ WRONG
InputStream stream = ClassLoader.getSystemResourceAsStream("metadata.json");

// ✅ RIGHT
InputStream stream = this.getClass().getClassLoader().getResourceAsStream("metadata.json");
```

**Why:** In OSGi, `getSystemClassLoader()` doesn't have access to bundle resources.

---

## 📊 Performance Considerations

### Issue: Slow Metadata Loading

**Expected:** 100ms-1s for typical metadata sets
**If Slower:** Check for:
- Excessive validation during loading
- Large metadata files (>10MB)
- Network-based URIs (use local files when possible)
- Recursive/circular dependencies

**Optimization:**
- Cache loaded metadata at application level
- Use file-based URIs instead of HTTP
- Lazy-load non-critical metadata

---

### Issue: High Memory Usage

**Expected:** 10-50MB for typical metadata sets
**If Higher:** Check for:
- Memory leaks (strong references in computed caches)
- Duplicate loaders (should have one loader per context)
- Excessive computed cache entries

**Monitoring:**
```java
// Add to HybridCache for monitoring
public int getCacheSize() {
    return modernCache.size() + legacyCache.size();
}
```

---

## 🧪 Testing Gotchas

### Gotcha: Tests Pass Individually, Fail Together

**Cause:** Static state not being reset between tests

**Solution:**
```java
@After
public void tearDown() {
    // Clear any static caches
    MetaDataCache.clear();
    MetaDataRegistry.reset(); // If available
}
```

---

### Gotcha: Constraint Tests Failing with "Name Required"

**Cause:** Constraint naming conventions not followed

**Solution:**
Ensure all constraints have format: `{domain}:{type}:{operation}`

Example:
```json
{
  "name": "metadata:field:placement",
  "type": "placement",
  ...
}
```

See [docs/constraints.md](docs/constraints.md) for naming conventions.

---

## 🔄 Migration Issues

### Migrating from v6.1.0 to v6.2.5

**Breaking Changes:**

1. **Field Type Changes:**
   - `ByteField`, `ShortField` removed → use `IntegerField`
   - Separate array types removed → use `@isArray: true`
   - Added `DecimalField` for high-precision decimals

   **Migration:**
   ```json
   // OLD
   { "field": { "type": "field", "subType": "byteArray", "name": "data" } }

   // NEW
   { "field": { "type": "field", "subType": "int", "name": "data", "@isArray": true } }
   ```

2. **Provider Registration:**
   - Direct `MetaDataRegistry` manipulation deprecated
   - Must implement `MetaDataTypeProvider`

   **Migration:**
   ```java
   // OLD (deprecated)
   MetaDataRegistry.getInstance().registerType(...);

   // NEW (required)
   public class MyProvider implements MetaDataTypeProvider {
       @Override
       public void registerTypes(MetaDataRegistry registry) {
           MyField.registerTypes(registry);
       }
   }
   ```

3. **Constraint Naming:**
   - All constraints must have format: `{domain}:{type}:{operation}`
   - Old constraints without this format will fail validation

---

## 📝 Documentation Gaps

**Areas that need more examples:**
- Complex MetaRelationship scenarios
- Custom MetaView implementations
- Advanced ObjectManager usage
- Performance tuning guide

**Status:** Low priority, framework is well-documented overall

---

## 🔮 Future Concerns

### Potential Issues to Watch

1. **JUnit 4 → JUnit 5 Migration**
   - Currently using JUnit 4.13.2
   - JUnit 5 has better features
   - Migration would be significant effort
   - **Priority:** LOW

2. **React/TypeScript Version Updates**
   - Web module uses React 18, TypeScript 5
   - Major version updates may require changes
   - **Priority:** MEDIUM - Monitor for breaking changes

3. **Java LTS Updates**
   - Currently Java 17 LTS (supported until 2029)
   - Java 21 LTS available (supported until 2031)
   - Migration straightforward but requires testing
   - **Priority:** LOW - Java 17 is fine for years

---

## 🆘 Getting Help

### If You Encounter an Issue Not Listed Here:

1. **Check existing docs:**
   - [ARCHITECTURE.md](ARCHITECTURE.md) - Design principles
   - [SETUP.md](SETUP.md) - Build and environment
   - [docs/*.md](docs/) - Topic-specific guides

2. **Search codebase:**
   ```bash
   grep -r "your error message" .
   ```

3. **Check tests:**
   - Look for similar test cases
   - Tests often show correct usage patterns

4. **Review provider implementations:**
   - `metadata/src/main/java/.../types/` - Provider examples
   - Shows correct patterns for registration

5. **Add issue here:**
   - Document the problem
   - Document the solution
   - Help future developers

---

**Remember:** Most issues stem from treating MetaObjects like a traditional data framework instead of a metadata framework analogous to Java reflection. When in doubt, think "ClassLoader" not "Hibernate".

---

**Last Updated:** 2025-11-15
