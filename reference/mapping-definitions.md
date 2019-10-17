# Mapping definitions

To specify how data from a service is mapped and transformed to a schema – and how to map and transform it back to the service – you set up a mapping definition, potentially with transform functions.

Note that mappings will be used to map data both from and to a service, and in the description below, we'll explain how this happens.

This is the full mapping definition:

```javascript
{
  id: <string>,
  schema: <schema id>,
  mapping: <mapping definition>
}
```

## Mapping defintion properties

### `id`

Optional unique `id` for a mapping definition. This is the `id` referenced from a service definition, but is not needed when a mapping is defined directly on the service.

### `schema`

The `id` of the target schema. Optional at this point, might be required in the future.

### `mapping`

This is the actual mapping definition, in the format of [MapTransform](https://github.com/integreat-io/map-transform), either a mutation object or a mutation pipeline.

&lt;examples&gt;

This mapping definition may reference transform functions and mapping pipelines. These are provided to `Integreat.create()` as `transforms` and `mappings`.

