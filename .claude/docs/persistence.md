# MetaObjects Persistence Patterns

**ObjectManager database integration for MetaObjects**

---

## Overview

ObjectManagerDB provides database persistence for MetaObjects-based applications with automatic table creation, CRUD operations, and query support.

**Key Principle:** Metadata-driven database mapping.

---

## Core Components

### ObjectManagerDB
- Main persistence interface
- Connection management
- CRUD operations
- Query execution

### MetaIdentity
- Defines primary keys
- Generation strategies
- Composite key support
- Replaces deprecated MetaKey

### Database Drivers
- **DerbyDriver** - Apache Derby (embedded/network)
- **PostgreSQLDriver** - PostgreSQL
- **MySQLDriver** - MySQL/MariaDB
- **OracleDriver** - Oracle Database

---

## MetaIdentity - MANDATORY for Persistence

### ⚠️ CRITICAL REQUIREMENT

**All persistent objects MUST have MetaIdentity metadata.**

Without it, ObjectManagerDB fails with "Attempt to modify an identity column" errors.

### Simple Primary Key

```json
{
  "object": {
    "name": "User",
    "subType": "managed",
    "@dbTable": "users",
    "children": [
      {
        "field": {
          "name": "id",
          "subType": "long",
          "@dbColumn": "user_id"
        }
      },
      {
        "identity": {
          "name": "user_pk",
          "subType": "primary",
          "fields": ["id"],
          "@generation": "increment"
        }
      }
    ]
  }
}
```

**Key Points:**
- `subType`: "primary" (required)
- `fields`: Array of field names
- `@generation`: How ID is generated

### Composite Primary Key

```json
{
  "object": {
    "name": "UserRole",
    "subType": "managed",
    "children": [
      {
        "field": {"name": "userId", "subType": "long"}
      },
      {
        "field": {"name": "roleId", "subType": "long"}
      },
      {
        "identity": {
          "name": "user_role_pk",
          "subType": "primary",
          "fields": ["userId", "roleId"],
          "@generation": "assigned"
        }
      }
    ]
  }
}
```

---

## Generation Strategies

### Auto-Increment

```json
{
  "identity": {
    "name": "pk",
    "subType": "primary",
    "fields": ["id"],
    "@generation": "increment"
  }
}
```

**Database mapping:**
- PostgreSQL: `SERIAL` or `IDENTITY`
- MySQL: `AUTO_INCREMENT`
- Oracle: `IDENTITY COLUMN` or sequence
- Derby: `GENERATED ALWAYS AS IDENTITY`

### UUID/GUID

```json
{
  "identity": {
    "name": "pk",
    "subType": "primary",
    "fields": ["id"],
    "@generation": "uuid"
  }
}
```

**Application-generated UUID before insert.**

### Assigned

```json
{
  "identity": {
    "name": "pk",
    "subType": "primary",
    "fields": ["id"],
    "@generation": "assigned"
  }
}
```

**Application must set ID before insert.**

### Natural Key (No Generation)

```json
{
  "identity": {
    "name": "pk",
    "subType": "primary",
    "fields": ["username"]
  }
}
```

**No `@generation` attribute - natural key.**

---

## Database Mapping Attributes

### Object Level

```json
{
  "object": {
    "name": "User",
    "@dbTable": "USERS",           // Table name
    "@dbSchema": "public",         // Schema name (optional)
    "@dbCatalog": "mydb"           // Catalog name (optional)
  }
}
```

### Field Level

```json
{
  "field": {
    "name": "email",
    "@dbColumn": "EMAIL_ADDRESS",  // Column name
    "@dbType": "VARCHAR(255)",     // SQL type override
    "@dbNullable": false,          // NOT NULL constraint
    "@dbUnique": true              // UNIQUE constraint
  }
}
```

### Identity Level

```json
{
  "identity": {
    "name": "pk",
    "@generation": "increment",
    "@dbIndexName": "pk_users",    // Index/constraint name
    "@dbClustered": true           // Clustered index (SQL Server)
  }
}
```

---

## ObjectManagerDB Setup

### Basic Configuration

```java
// 1. Initialize MetaData Loader
FileMetaDataLoader loader = new FileMetaDataLoader(options, "myLoader");
loader.init();
loader.register(); // Register for MetaDataUtil discovery

// 2. Initialize Database
EmbeddedDataSource dataSource = new EmbeddedDataSource();
dataSource.setDatabaseName("memory:testdb");
dataSource.setCreateDatabase("create");

// 3. Initialize ObjectManagerDB
ObjectManagerDB objectManager = new ObjectManagerDB();
objectManager.setDriverClass("com.metaobjects.manager.db.driver.DerbyDriver");
objectManager.setDataSource(dataSource);

// 4. Auto-create tables
MetaClassDBValidatorService validator = new MetaClassDBValidatorService();
validator.setObjectManager(objectManager);
validator.setAutoCreate(true);

MetaDataLoaderRegistry registry = new MetaDataLoaderRegistry(ServiceRegistryFactory.getDefault());
registry.registerLoader(loader);
validator.setMetaDataLoaderRegistry(registry);
validator.init();
```

---

## CRUD Operations

### Create

```java
ObjectConnection connection = objectManager.getConnection();
try {
    User user = new User();
    user.setMetaData(loader.getMetaObjectByName("User")); // REQUIRED!
    user.setUsername("john");
    user.setEmail("john@example.com");

    objectManager.createObject(connection, user);
    // Auto-increment ID assigned by database
    System.out.println("Created user with ID: " + user.getId());
} finally {
    connection.close();
}
```

### Read

```java
ObjectConnection connection = objectManager.getConnection();
try {
    User user = (User) objectManager.getObject(connection, User.class, userId);
    System.out.println("Found: " + user.getUsername());
} finally {
    connection.close();
}
```

### Update

```java
ObjectConnection connection = objectManager.getConnection();
try {
    User user = (User) objectManager.getObject(connection, User.class, userId);
    user.setEmail("newemail@example.com");
    objectManager.updateObject(connection, user);
} finally {
    connection.close();
}
```

### Delete

```java
ObjectConnection connection = objectManager.getConnection();
try {
    User user = (User) objectManager.getObject(connection, User.class, userId);
    objectManager.destroyObject(connection, user);
} finally {
    connection.close();
}
```

---

## Query Patterns

### By Primary Key

```java
User user = (User) objectManager.getObject(connection, User.class, userId);
```

### By Query

```java
QueryBuilder query = objectManager.getQueryBuilder(User.class);
query.where("email = ?", "john@example.com");

List<User> users = objectManager.query(connection, query);
```

### Count

```java
long count = objectManager.count(connection, User.class);
```

---

## Database Overlay Pattern

**Separate database config from core metadata:**

### base-metadata.json
```json
{
  "object": {
    "name": "Store",
    "subType": "managed",
    "children": [
      {"field": {"name": "id", "subType": "long"}},
      {"identity": {"name": "store_pk", "subType": "primary", "fields": ["id"]}}
    ]
  }
}
```

### db-overlay.json
```json
{
  "object": {
    "name": "Store",
    "@dbTable": "STORE",
    "children": [
      {"field": {"name": "id", "@dbColumn": "ID"}},
      {"identity": {"name": "store_pk", "@generation": "increment", "@dbIndexName": "pk_store"}}
    ]
  }
}
```

**Load both:**
```java
loader.addSourceURI(URI.create("base-metadata.json"));
loader.addSourceURI(URI.create("db-overlay.json")); // Overlays attributes
```

---

## Common Patterns

### ❌ INCORRECT - Deprecated Patterns

**DON'T use @isPrimaryKey:**
```json
{"field": {"name": "id", "@isPrimaryKey": true}}  // ❌ Deprecated
```

**DON'T use MetaKey:**
```json
{"key": {"name": "primary", "@keys": ["id"]}}  // ❌ Use MetaIdentity
```

**DON'T use field-level @autoIncrementStrategy:**
```json
{"field": {"name": "id", "@autoIncrementStrategy": "sequential"}}  // ❌ Use MetaIdentity @generation
```

### ✅ CORRECT - Current Pattern

**Use MetaIdentity:**
```json
{
  "identity": {
    "name": "pk",
    "subType": "primary",
    "fields": ["id"],
    "@generation": "increment"
  }
}
```

---

## Critical Requirements

### Before Persistence Operations

1. **MetaIdentity Required**
   - All objects MUST have MetaIdentity
   - Must be `subType="primary"`
   - Must specify `@generation` for auto-generated IDs

2. **setMetaData() Required**
   ```java
   user.setMetaData(loader.getMetaObjectByName("User"));
   ```

3. **Registry Connection**
   - Tests MUST use MetaDataLoaderRegistry
   - Connects services to loaders

---

## Database Type Mappings

| MetaObjects | PostgreSQL | MySQL | SQL Server | Oracle |
|-------------|-----------|-------|------------|--------|
| string | VARCHAR | VARCHAR | VARCHAR | VARCHAR2 |
| int | INTEGER | INT | INT | NUMBER(10) |
| long | BIGINT | BIGINT | BIGINT | NUMBER(19) |
| float | REAL | FLOAT | REAL | FLOAT(24) |
| double | DOUBLE PRECISION | DOUBLE | FLOAT | FLOAT(53) |
| decimal | DECIMAL(19,2) | DECIMAL(19,2) | DECIMAL(19,2) | NUMBER(19,2) |
| boolean | BOOLEAN | TINYINT(1) | BIT | NUMBER(1) |
| date | DATE | DATE | DATE | DATE |
| timestamp | TIMESTAMP | DATETIME | DATETIME2 | TIMESTAMP |
| binary | BYTEA | BLOB | VARBINARY | BLOB |

---

## Testing

### Example Test

```java
@Test
public void testPersistence() throws Exception {
    ObjectConnection conn = objectManager.getConnection();
    try {
        // Create
        User user = new User();
        user.setMetaData(loader.getMetaObjectByName("User"));
        user.setUsername("test");
        objectManager.createObject(conn, user);

        Long id = user.getId();
        assertNotNull(id);

        // Read
        User loaded = (User) objectManager.getObject(conn, User.class, id);
        assertEquals("test", loaded.getUsername());

        // Update
        loaded.setUsername("updated");
        objectManager.updateObject(conn, loaded);

        // Verify
        User updated = (User) objectManager.getObject(conn, User.class, id);
        assertEquals("updated", updated.getUsername());

        // Delete
        objectManager.destroyObject(conn, updated);

        // Verify deleted
        User deleted = (User) objectManager.getObject(conn, User.class, id);
        assertNull(deleted);
    } finally {
        conn.close();
    }
}
```

---

## See Also

- [type-system.md](type-system.md) - Field types and mappings
- [testing.md](testing.md) - Unit test patterns
- [SETUP.md](../SETUP.md) - Building and running tests

---

**Last Updated:** 2025-11-15
