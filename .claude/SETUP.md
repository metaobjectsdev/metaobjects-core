# MetaObjects Core - Build & Setup Guide

**Complete guide to building, testing, and developing MetaObjects Core**

---

## Prerequisites

- **Java 21** (JDK 21 or higher)
- **Maven 3.8+**
- **Git** for version control
- **IDE** (IntelliJ IDEA or Eclipse recommended)

---

## Quick Start

### Clone and Build
```bash
cd ~/Development/metaobjects-core
mvn clean compile      # Compile all 10 modules
mvn test              # Run all tests
mvn package           # Build JARs
```

### Verify Build
```bash
mvn clean test
# Should show:  BUILD SUCCESS
# All modules: metadata, codegen, core, maven-plugin, objectmanager, etc.
```

---

## Key Build Commands

### Full Build Cycle
```bash
# Clean and build entire project (all 10 modules)
mvn clean compile

# Full test suite execution
mvn test

# Complete package with code generation
mvn package

# Install to local Maven repository
mvn install
```

### Targeted Builds
```bash
# Generate MetaDataFile schemas (automatic during package)
cd core && mvn metaobjects:generate@gen-schemas

# Run constraint system tests specifically
cd metadata && mvn test -Dtest=ConstraintSystemTest

# Run Maven plugin tests
cd maven-plugin && mvn test

# Build specific module (respects dependency order)
cd metadata && mvn compile
cd codegen && mvn compile
cd core && mvn compile
```

### Clean Operations
```bash
# Clean all build artifacts
mvn clean

# Clean specific module
cd metadata && mvn clean
```

---

## Build Status

### ✅ Latest Verification Results

**Clean Build:** All artifacts removed successfully
**Full Compilation:** All 10 modules compiled without errors
**Constraint Tests:** 8/8 tests passing with proper naming enforcement
**Maven Plugin:** 4/4 tests passing with ServiceLoader discovery
**Schema Generation:** Both JSON and XSD schemas generated correctly
**Package Build:** All modules packaged successfully

### Reactor Summary
```
[INFO] Reactor Summary for MetaObjects 6.2.5-SNAPSHOT:
[INFO] MetaObjects ........................................ SUCCESS
[INFO] MetaObjects :: MetaData ............................ SUCCESS
[INFO] MetaObjects :: Code Generation ..................... SUCCESS
[INFO] MetaObjects :: Core ................................ SUCCESS
[INFO] MetaObjects :: Maven Plugin ........................ SUCCESS
[INFO] MetaObjects :: ObjectManager ....................... SUCCESS
[INFO] MetaObjects :: ObjectManager :: RDB ................ SUCCESS
[INFO] MetaObjects :: ObjectManager :: NoSQL .............. SUCCESS
[INFO] MetaObjects :: Web ................................. SUCCESS
[INFO] MetaObjects :: Demo ................................ SUCCESS
[INFO] BUILD SUCCESS
```

---

## Module Dependency Order

MetaObjects uses a strict module dependency hierarchy:

```
1. metadata          # Base metadata models (no dependencies)
2. codegen           # Code generation (depends on metadata)
3. core              # Core framework (depends on metadata, codegen)
4. maven-plugin      # Maven tooling (depends on core)
5. objectmanager     # Persistence base (depends on core)
6. objectmanager-rdb # RDB support (depends on objectmanager)
7. objectmanager-nosql # NoSQL support (depends on objectmanager)
8. web               # Web integration (depends on core)
9. demo              # Demos (depends on web, objectmanager)
10. archetype        # Project templates
```

**Building individual modules:** Maven automatically builds dependencies first.

---

## Maven Configuration

### Parent POM
- All modules inherit from parent `pom.xml`
- Dependency versions managed centrally
- Consistent build configuration across modules

### Compiler Settings
```xml
<maven.compiler.source>17</maven.compiler.source>
<maven.compiler.target>17</maven.compiler.target>
<maven.compiler.release>17</maven.compiler.release>
```

### OSGi Bundle Support
- Enabled via Apache Felix Maven Bundle Plugin
- Generates OSGi manifests automatically
- Bundle-SymbolicName and Export-Package configured

### Distribution Profiles
- **default (Draagon):** Internal Draagon repository
- **nexus (Maven Central):** Maven Central repository

---

## Critical Files for Development

### Source Code Locations

**Core Framework:**
- `core/src/main/java/com/draagon/meta/` - Core implementation
- `metadata/src/main/java/com/draagon/meta/` - Metadata models
- `codegen/src/main/java/com/draagon/meta/` - Code generation

**Persistence:**
- `objectmanager/src/main/java/com/draagon/meta/object/` - Base persistence
- `objectmanager-rdb/src/main/java/com/draagon/meta/object/rdb/` - Relational DB
- `objectmanager-nosql/src/main/java/com/draagon/meta/object/nosql/` - NoSQL DB

**Web & React:**
- `web/src/typescript/components/metaviews/` - React MetaView components
- `web/src/typescript/components/forms/` - React form components
- `web/src/typescript/types/metadata.ts` - TypeScript types
- `web/src/main/java/com/draagon/meta/web/react/api/` - Spring controllers

**Demo Application:**
- `demo/src/main/java/com/draagon/meta/demo/fishstore/api/` - Demo controllers
- `demo/src/main/resources/metadata/fishstore-metadata.json` - Demo metadata

**Tests:**
- `*/src/test/java/` - JUnit tests in each module
- `*/src/test/resources/` - Test resources

### Configuration Files

**Maven:**
- `pom.xml` - Parent POM (root)
- `*/pom.xml` - Module POMs
- `maven-plugin/src/main/resources/META-INF/maven/plugin.xml` - Plugin descriptor

**React/TypeScript:**
- `web/package.json` - NPM dependencies
- `web/tsconfig.json` - TypeScript configuration
- `web/webpack.config.js` - Webpack bundler

**OSGi:**
- Generated in `META-INF/MANIFEST.MF` by Felix plugin

**Service Discovery:**
- `META-INF/services/com.draagon.meta.loader.types.MetaDataTypeProvider` - ServiceLoader registration

**Metadata:**
- `*/src/main/resources/metadata/*.json` - JSON metadata definitions
- `*/src/main/resources/metadata/*.xsd` - XSD schemas

### Documentation

- `README.md` - Main project README
- `RELEASE_NOTES.md` - Version history
- `.claude/` - Claude Code documentation (this directory)

---

## Key Technologies & Dependencies

### Core Java Stack
- **Java 21** with Maven compiler plugin 3.13.0
- **SLF4J 2.0.16** + **Logback 1.5.16** for logging
- **JUnit 4.13.2** for testing
- **Gson 2.13.1** for JSON handling
- **Commons Validator 1.9.0** for validation

### OSGi & Services
- **Apache Felix Maven Bundle Plugin 5.1.9** for OSGi bundles
- **ServiceLoader** for provider discovery

### Frontend (Web Module)
- **React 18+** for UI components
- **TypeScript 5+** for type-safe JavaScript
- **Redux Toolkit** for state management
- **Webpack 5** for bundling

### Build Tools
- **Maven 3.8+** (required)
- **maven-compiler-plugin 3.13.0**
- **maven-surefire-plugin 3.5.2** for testing
- **metaobjects-maven-plugin** for schema generation

---

## IDE Setup

### IntelliJ IDEA (Recommended)

1. **Import Project:**
   - File → Open → Select `metaobjects-core/pom.xml`
   - Choose "Open as Project"
   - IntelliJ auto-detects Maven structure

2. **Set JDK:**
   - File → Project Structure → Project
   - Set SDK to Java 21
   - Set language level to 17

3. **Enable Annotation Processing:**
   - Settings → Build, Execution, Deployment → Compiler → Annotation Processors
   - Check "Enable annotation processing"

4. **Install Plugins:**
   - Lombok (if using)
   - Maven Helper
   - OSGi (optional)

5. **Run Configurations:**
   - Create "Maven" run configuration
   - Working directory: `$ProjectFileDir$`
   - Command line: `clean test`

### Eclipse

1. **Import Maven Project:**
   - File → Import → Maven → Existing Maven Projects
   - Select `metaobjects-core` directory

2. **Set JDK:**
   - Window → Preferences → Java → Installed JREs
   - Add JDK 21 if not present
   - Project → Properties → Java Build Path → Set to JDK 21

3. **Install Plugins:**
   - M2Eclipse (Maven integration)
   - Eclipse Checkstyle Plugin (optional)

### VS Code

1. **Install Extensions:**
   - Extension Pack for Java (Microsoft)
   - Maven for Java
   - Spring Boot Extension Pack (if working with web module)

2. **Open Folder:**
   - Open `metaobjects-core` directory
   - VS Code auto-detects Maven project

3. **Configure Java:**
   - Set `java.configuration.runtimes` to Java 21 in settings.json

---

## Testing

### Run All Tests
```bash
mvn test
```

### Run Specific Module Tests
```bash
cd metadata && mvn test
cd core && mvn test
```

### Run Specific Test Class
```bash
cd metadata && mvn test -Dtest=ConstraintSystemTest
cd maven-plugin && mvn test -Dtest=ServiceLoaderTest
```

### Run Single Test Method
```bash
mvn test -Dtest=ConstraintSystemTest#testPlacementConstraints
```

### Skip Tests
```bash
mvn package -DskipTests
```

### Test Coverage
```bash
# Run tests with coverage (if jacoco plugin configured)
mvn clean test jacoco:report
```

See [docs/testing.md](docs/testing.md) for detailed testing guidelines.

---

## Schema Generation

The MetaObjects Maven Plugin generates JSON and XSD schemas from metadata definitions.

### Automatic Generation
```bash
cd core
mvn package  # Schemas generated automatically
```

### Manual Generation
```bash
cd core
mvn metaobjects:generate@gen-schemas
```

### Generated Files
- `core/target/generated-resources/metadata/*.json`
- `core/target/generated-resources/metadata/*.xsd`

---

## Common Development Tasks

### Add New MetaData Type

1. **Create Concrete Class:**
   ```java
   // metadata/src/main/java/com/draagon/meta/field/MyField.java
   public class MyField extends MetaField {
       public MyField(String name) {
           super("mytype", name);
       }

       public static void registerTypes(MetaDataRegistry registry) {
           registry.registerType(MyField.class, def -> def
               .type("field").subType("mytype")
               .description("My custom field type")
               .inheritsFrom("metadata", "field")
           );
       }
   }
   ```

2. **Register in Provider:**
   ```java
   // metadata/src/main/java/.../MyTypesMetaDataProvider.java
   public class MyTypesMetaDataProvider implements MetaDataTypeProvider {
       @Override
       public void registerTypes(MetaDataRegistry registry) {
           MyField.registerTypes(registry);
       }

       @Override
       public int getPriority() { return 30; }
   }
   ```

3. **Add ServiceLoader Registration:**
   ```
   # metadata/src/main/resources/META-INF/services/
   # com.draagon.meta.loader.types.MetaDataTypeProvider

   com.draagon.meta.field.MyTypesMetaDataProvider
   ```

4. **Test:**
   ```bash
   cd metadata && mvn test
   ```

### Update Dependency Version

1. Edit parent `pom.xml`:
   ```xml
   <properties>
       <gson.version>2.13.1</gson.version>
   </properties>
   ```

2. Rebuild:
   ```bash
   mvn clean install
   ```

### Add Module to Reactor

1. Create module directory with `pom.xml`
2. Add to parent `pom.xml`:
   ```xml
   <modules>
       <module>my-new-module</module>
   </modules>
   ```
3. Set parent in module `pom.xml`
4. Rebuild reactor:
   ```bash
   mvn clean install
   ```

---

## Troubleshooting

### Build Fails with "Cannot find symbol"
**Cause:** Module dependencies not resolved
**Fix:**
```bash
mvn clean install  # Install dependencies to local repo
```

### Tests Fail with ServiceLoader Issues
**Cause:** Missing `META-INF/services/` entries
**Fix:** Verify provider files exist in `src/main/resources/META-INF/services/`

### OSGi Manifest Generation Fails
**Cause:** Felix plugin configuration issue
**Fix:** Check `<packaging>bundle</packaging>` in module `pom.xml`

### Schema Generation Fails
**Cause:** Core module not built
**Fix:**
```bash
cd core && mvn clean install
```

### IDE Can't Find Classes
**Cause:** Maven project not imported correctly
**Fix:** Reimport Maven project or run "Maven → Reload Project"

---

## Publishing to Maven Central

### Prerequisites
- GPG key for signing
- Sonatype OSSRH account
- Configure `~/.m2/settings.xml` with credentials

### Build with Nexus Profile
```bash
mvn clean deploy -P nexus
```

### Steps
1. `mvn clean verify` - Verify build
2. `mvn deploy -P nexus` - Deploy to staging
3. Login to https://oss.sonatype.org
4. Close staging repository
5. Release to Maven Central

See Maven Central Publishing docs for details.

---

## Next Steps

- Review [ARCHITECTURE.md](ARCHITECTURE.md) for design principles
- Read [docs/testing.md](docs/testing.md) for testing best practices
- Check [KNOWN-ISSUES.md](KNOWN-ISSUES.md) for common pitfalls
- See [CONTEXT.md](CONTEXT.md) for recent changes

---

**Last Updated:** 2025-11-15
