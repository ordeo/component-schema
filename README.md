# Ordeo component schema

JSON Schemas for the platform-neutral component descriptions used by Ordeo.

Published at <https://ordeo.github.io/component-schema/>.

## Using a schema

Reference the versioned URL from a component file, and your IDE validates it:

```json
{
  "$schema": "https://ordeo.github.io/component-schema/component/v1.schema.json",
  "components": [
    ...
  ]
}
```

## Layout

| Path                              | Purpose                                          |
|-----------------------------------|--------------------------------------------------|
| `schemas/<name>/v<N>.schema.json` | The schemas. One file per major version.         |
| `examples/`                       | Component files that validate against the schema |
