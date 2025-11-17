# React Integration & MetaView Components

**React/TypeScript integration for metadata-driven UIs**

---

## Overview

MetaObjects includes React components that automatically generate UIs from metadata definitions.

**Key Principle:** Metadata drives UI generation.

---

## MetaView System

### What is MetaView?

**MetaView** components render metadata as interactive UI elements:
- Forms
- Tables
- Lists
- Detail views

**Driven by:** MetaObject metadata definitions

---

## Project Structure

### Web Module

```
web/
├── src/
│   ├── typescript/
│   │   ├── components/
│   │   │   ├── metaviews/       # MetaView components
│   │   │   └── forms/           # Form components
│   │   ├── types/
│   │   │   └── metadata.ts      # TypeScript types
│   │   └── store/               # Redux store
│   ├── main/java/
│   │   └── com/draagon/meta/web/react/api/
│   │       └── MetadataController.java
│   └── main/resources/
│       └── metadata/
│           └── web-metadata.json
├── package.json
├── tsconfig.json
└── webpack.config.js
```

---

## Core Components

### MetaObjectView

**Renders complete object view:**

```typescript
import { MetaObjectView } from './components/metaviews';

<MetaObjectView
  metaObjectName="User"
  objectId={userId}
  mode="view"
/>
```

**Props:**
- `metaObjectName`: Name of MetaObject
- `objectId`: ID of object instance
- `mode`: "view" | "edit" | "create"

### MetaFieldView

**Renders individual field:**

```typescript
import { MetaFieldView } from './components/metaviews';

<MetaFieldView
  field={metaField}
  value={fieldValue}
  onChange={handleChange}
/>
```

**Automatically renders:**
- String → Text input
- Int/Long → Number input
- Boolean → Checkbox
- Date → Date picker
- Object reference → Dropdown

### MetaFormView

**Renders complete form:**

```typescript
import { MetaFormView } from './components/forms';

<MetaFormView
  metaObjectName="User"
  onSubmit={handleSubmit}
  initialValues={userData}
/>
```

---

## TypeScript Types

### Metadata Types

```typescript
// types/metadata.ts

export interface MetaObject {
  name: string;
  type: string;
  subType?: string;
  fields: MetaField[];
  attributes: Record<string, any>;
}

export interface MetaField {
  name: string;
  type: string;
  subType: string;
  label?: string;
  required?: boolean;
  validators?: MetaValidator[];
}

export interface MetaValidator {
  name: string;
  type: string;
  message?: string;
  params?: Record<string, any>;
}
```

---

## Spring Boot Integration

### API Endpoints

```java
@RestController
@RequestMapping("/api/metadata")
public class MetadataController {

    @GetMapping("/{objectName}")
    public MetaObject getMetadata(@PathVariable String objectName) {
        return metaDataLoader.getMetaObjectByName(objectName);
    }

    @GetMapping("/{objectName}/{id}")
    public Object getObject(
        @PathVariable String objectName,
        @PathVariable Long id
    ) {
        return objectManager.getObject(connection, objectName, id);
    }

    @PostMapping("/{objectName}")
    public Object createObject(
        @PathVariable String objectName,
        @RequestBody Map<String, Object> data
    ) {
        // Create object from metadata + data
    }
}
```

---

## Redux Integration

### State Management

```typescript
// store/metadataSlice.ts
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const fetchMetadata = createAsyncThunk(
  'metadata/fetch',
  async (objectName: string) => {
    const response = await fetch(`/api/metadata/${objectName}`);
    return response.json();
  }
);

const metadataSlice = createSlice({
  name: 'metadata',
  initialState: {
    objects: {},
    loading: false,
    error: null
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchMetadata.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchMetadata.fulfilled, (state, action) => {
        state.objects[action.payload.name] = action.payload;
        state.loading = false;
      });
  }
});
```

---

## Example Usage

### Complete Form

```typescript
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { MetaFormView } from './components/forms';
import { fetchMetadata } from './store/metadataSlice';

export const UserForm: React.FC = () => {
  const dispatch = useDispatch();
  const metadata = useSelector(state => state.metadata.objects['User']);

  useEffect(() => {
    dispatch(fetchMetadata('User'));
  }, [dispatch]);

  const handleSubmit = async (values) => {
    await fetch('/api/metadata/User', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(values)
    });
  };

  if (!metadata) return <div>Loading...</div>;

  return (
    <MetaFormView
      metaObject={metadata}
      onSubmit={handleSubmit}
    />
  );
};
```

---

## Field Type Rendering

### Automatic Field Rendering

**Based on field subType:**

| subType | Component | Notes |
|---------|-----------|-------|
| string | `<input type="text">` | Standard text input |
| int, long | `<input type="number">` | Number input |
| float, double, decimal | `<input type="number" step="0.01">` | Decimal input |
| boolean | `<input type="checkbox">` | Checkbox |
| date | `<input type="date">` | Date picker |
| timestamp | `<input type="datetime-local">` | DateTime picker |
| object | `<select>` | Dropdown with object list |

### Custom Renderers

```typescript
const customRenderers = {
  email: (props) => <input type="email" {...props} />,
  phone: (props) => <input type="tel" {...props} />,
  currency: (props) => <CurrencyInput {...props} />
};

<MetaFieldView
  field={field}
  value={value}
  customRenderers={customRenderers}
/>
```

---

## Building & Deployment

### Build React Components

```bash
cd ~/Development/metaobjects-core/web
npm install
npm run build
```

**Output:** `web/target/classes/static/`

### Include in Spring Boot

```xml
<!-- pom.xml -->
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>exec-maven-plugin</artifactId>
            <executions>
                <execution>
                    <id>npm-install</id>
                    <goals><goal>exec</goal></goals>
                    <phase>generate-resources</phase>
                    <configuration>
                        <executable>npm</executable>
                        <arguments><argument>install</argument></arguments>
                    </configuration>
                </execution>
                <execution>
                    <id>npm-build</id>
                    <goals><goal>exec</goal></goals>
                    <phase>generate-resources</phase>
                    <configuration>
                        <executable>npm</executable>
                        <arguments><argument>run</argument><argument>build</argument></arguments>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

---

## Demo Application

### FishStore Demo

**Location:** `demo/src/main/java/com/draagon/meta/demo/fishstore/`

**Features:**
- Complete CRUD operations
- MetaView component usage
- Spring Boot + React integration
- PostgreSQL persistence

**Running:**
```bash
cd ~/Development/metaobjects-core/demo
mvn spring-boot:run
```

**Access:** http://localhost:8080

---

## Future Enhancements

### Planned Components

- **MetaTableView** - Sortable, filterable tables
- **MetaSearchView** - Advanced search forms
- **MetaChartView** - Metadata-driven charts
- **MetaWizardView** - Multi-step wizards

### TypeScript Improvements

- Stricter type definitions
- Better validation integration
- Performance optimizations

---

## See Also

- [type-system.md](type-system.md) - Field types and mappings
- [persistence.md](persistence.md) - Backend data integration
- [SETUP.md](../SETUP.md) - Building web module

---

**Last Updated:** 2025-11-15
