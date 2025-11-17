# MetaObjects Core - Claude Code Guide

**Universal metadata-driven development platform for enterprise AI at scale**

---

## Quick Reference

| Topic | File | Description |
|-------|------|-------------|
| **Architecture** | [ARCHITECTURE.md](ARCHITECTURE.md) | Core design principles, patterns, OSGI compatibility |
| **Setup & Build** | [SETUP.md](SETUP.md) | Maven commands, IDE setup, dependencies |
| **Current Work** | [CONTEXT.md](CONTEXT.md) | Recent changes, what's in progress, session notes |
| **Known Issues** | [KNOWN-ISSUES.md](KNOWN-ISSUES.md) | Bugs, limitations, lessons learned |
| **Type System** | [docs/type-system.md](docs/type-system.md) | AI-optimized field types, cross-language mapping |
| **Constraints** | [docs/constraints.md](docs/constraints.md) | Fluent constraint system, validation patterns |
| **Persistence** | [docs/persistence.md](docs/persistence.md) | ObjectManager, database integration patterns |
| **Testing** | [docs/testing.md](docs/testing.md) | Unit test setup, best practices |
| **React Integration** | [docs/react-integration.md](docs/react-integration.md) | MetaView components, TypeScript integration |

---

## Project Overview

**MetaObjects Core** is a Java-based metadata framework providing sophisticated control over applications beyond traditional model-driven development.

- **Version:** 6.2.5-SNAPSHOT (Maven Central Ready)
- **Java:** 17 LTS (Production Ready)
- **Build:** Maven
- **License:** Apache License 2.0
- **URL:** https://metaobjects.com

### What is MetaObjects?

MetaObjects is analogous to Java's reflection system, but for runtime metadata:

| Java Reflection | MetaObjects Framework |
|----------------|----------------------|
| `Class.forName()` | `loader.getMetaObjectByName()` |
| `Class.getFields()` | `metaObject.getMetaFields()` |
| `Field.get(object)` | `metaField.getValue(object)` |
| Permanent in memory | Permanent MetaData cache |
| Thread-safe reads | Thread-safe metadata access |

**Key Principle:** Read-optimized with controlled mutability (99.9% reads, rare updates)

---

## Quick Start

### Build Entire Project
```bash
cd ~/Development/metaobjects-core
mvn clean compile      # Compile all modules
mvn test              # Run all tests
mvn package           # Build JARs
```

### Build Specific Module
```bash
cd metadata && mvn compile
cd core && mvn compile
```

### Run Tests
```bash
mvn test                                    # All tests
cd metadata && mvn test -Dtest=ConstraintSystemTest  # Specific test
```

---

## Project Structure

```
metaobjects-core/
├── metadata/          # Core metadata models (MetaObject, MetaField, etc.)
├── codegen/          # Code generation utilities
├── core/             # Core framework implementation
├── maven-plugin/     # Maven plugin for schema generation
├── objectmanager/    # Persistence layer base
├── objectmanager-rdb/   # Relational database support
├── objectmanager-nosql/ # NoSQL database support
├── web/              # Web integration (React, Spring)
├── demo/             # Demo applications (FishStore)
└── archetype/        # Maven archetype for new projects
```

**Total:** 10 Maven modules

---

## Module Dependency Order

Build respects this dependency hierarchy:
1. `metadata` - Base metadata models
2. `codegen` - Code generation (depends on metadata)
3. `core` - Core framework (depends on metadata, codegen)
4. `maven-plugin` - Maven tooling (depends on core)
5. `objectmanager` - Persistence base (depends on core)
6. `objectmanager-rdb` - RDB support (depends on objectmanager)
7. `objectmanager-nosql` - NoSQL support (depends on objectmanager)
8. `web` - Web integration (depends on core)
9. `demo` - Demos (depends on web, objectmanager)
10. `archetype` - Project templates

---

## Recent Major Achievements

### ✅ AI-Optimized Type System (v6.3.0+)
- **33% complexity reduction** in field types
- 8 fragmented types → 6 core semantic types
- Universal `@isArray` modifier eliminates type explosion
- Cross-language mapping: Java, C#, TypeScript, Node.js
- See [docs/type-system.md](docs/type-system.md) for details

### ✅ Maven Central Publishing Ready (v6.2.5)
- Complete POM metadata
- Source/Javadoc JARs configured
- Nexus deployment profile ready
- Apache License 2.0

### ✅ Comprehensive Modernization (2024-2025)
- Java 17 LTS migration
- Constraint system refactoring
- Test coverage improvements
- ServiceLoader resolution
- See [CONTEXT.md](CONTEXT.md) for details

---

## Core Technologies

- **Java 21** (source/target: 17 LTS)
- **Maven 3.8+**
- **SLF4J + Logback** (logging)
- **JUnit 4.13.2** (testing)
- **Gson 2.13.1** (JSON)
- **React + TypeScript** (web UI)
- **Redux Toolkit** (state management)
- **OSGi** compatible (Apache Felix)

---

## Development Workflow

### 1. Make Changes
```bash
# Edit files in appropriate module
vim metadata/src/main/java/com/draagon/meta/...
```

### 2. Test Locally
```bash
cd metadata
mvn test
```

### 3. Build & Verify
```bash
cd ~/Development/metaobjects-core
mvn clean package
```

### 4. Generate Schemas (if metadata changed)
```bash
cd core
mvn metaobjects:generate@gen-schemas
```

---

## Critical Architectural Principles

### ⚠️ READ-OPTIMIZED DESIGN

MetaObjects is **NOT** a typical data access pattern. It's analogous to Java's ClassLoader:

1. **Heavy startup, fast runtime** - Metadata loaded once at startup
2. **Permanent in memory** - Like `Class` objects, stays loaded
3. **Thread-safe reads** - No synchronization needed (99.9% of operations)
4. **Rare controlled updates** - Copy-on-Write for dynamic changes
5. **OSGI compatible** - WeakReference caching for bundle unload safety

**DO NOT** treat MetaData objects as frequently mutable domain objects!

See [ARCHITECTURE.md](ARCHITECTURE.md) for deep dive.

---

## Getting Help

### Documentation Navigation

**Start here:**
1. Read this README for overview
2. Check [CONTEXT.md](CONTEXT.md) for recent work
3. Review [ARCHITECTURE.md](ARCHITECTURE.md) for design principles
4. Follow [SETUP.md](SETUP.md) for build details

**Specific topics:**
- Type system design → [docs/type-system.md](docs/type-system.md)
- Validation → [docs/constraints.md](docs/constraints.md)
- Database integration → [docs/persistence.md](docs/persistence.md)
- Testing → [docs/testing.md](docs/testing.md)
- React/TypeScript → [docs/react-integration.md](docs/react-integration.md)

### Known Issues & Lessons

Check [KNOWN-ISSUES.md](KNOWN-ISSUES.md) for:
- OSGI compatibility pitfalls (avoid these mistakes!)
- ServiceLoader resolution patterns
- Testing gotchas
- Performance considerations

---

## Next Steps

### For New Claude Code Sessions

1. Read [CONTEXT.md](CONTEXT.md) - See what's currently in progress
2. Check [KNOWN-ISSUES.md](KNOWN-ISSUES.md) - Avoid known pitfalls
3. Review relevant docs/ files for your task
4. **Update CONTEXT.md** at end of session with changes made

### For Development Work

1. Understand the architecture ([ARCHITECTURE.md](ARCHITECTURE.md))
2. Set up build environment ([SETUP.md](SETUP.md))
3. Read topic-specific docs as needed
4. Run tests after changes
5. Update CONTEXT.md with progress

---

**Last Updated:** 2025-11-15
**Maintainer:** Doug Mealing (doug@dougmealing.com)
**Organization:** Doug Mealing LLC dba MetaObjects
**License:** Apache License 2.0
