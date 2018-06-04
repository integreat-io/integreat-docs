# Mapping definitions

To specify how data from a service is mapped and transformed to a schema – and how to map and transform it back to the service – you set up a mapping definition, potentially with transform and mutate functions.

This is the full mapping definition:

```javascript
{
  id: <string>,
  path: <path string>,
  attributes: {
    <attribute id>: {
      path: <path string>,
      transform: <transformers pipeline>
    },
    ...
  },
  relationships: {
    <relationship id>: {
      path: <path string>,
      transform: <transformers pipeline>
    },
    ...
  },
  qualifier: <qualifier string>,
  mutate: <mutaters pipeline>,
  filterFrom: <filters pipeline>,
  filterTo: <filters pipeline>
}
```

## Mapping defintion properties

### `id`

Optional unique `id` for a mapping definition. This is the `id` referenced from a service definition, but is not needed when a mapping is defined directly on the service.

### `path`

A path into the service data, in Integreat's extended dot notation. The path on the top level of a mapping should point to where the data for each item is, which will typically be an array – at least when the data contains a collection. For example `data.items[]`.

Attributes and relationships have their own `path` properties, which will start from the point the mapping `path` points to.

See the Path format for more on how to specify a path.

### `attributes` and `relationships`

The fields of a schema is separated into `attributes` and `relationships`, and this distinction is used in mappings too, even though they are defined in the exact same way. Put simply, when a field references data items with `id` and `type` \(schema\) it's a relationship – anything else is an attribute. When mapping to a relationship, the value mapped value will be the id of the relationship. For attributes, the value is the value.

Both `attributes` and `relationships` may be defined in a short form, with a path string instead of a field definition object. `title: 'segment.headline'` is a shorter form of `title: {path: 'segments.headline'}`. This is handy when you don't need to define a `transform` pipeline.

#### `path`

The path property on `attributes` and `relationships` points to the actual value to be used for the field. If a `transform` pipeline is specified, the value will be sent through the pipeline before it is cased to the type specified in the schema. The `path` is relative to the `path` defined on the mapping itself.

#### `transform`

A `transform` pipeline defines one or more transformators that the mapped value will be passed through before being cast with the relevant type according to the schema. Transformers are functions, or objects with `to` and `from` functions, that will accept a value and return a value, and everything in between is up to the transformator function.

If you, as an example, have a transformator that turns a string to uppercase and another tranformator that returns only the first word in a string, you might specify an attribute like this:

```javascript
bigWord: {
  path: 'content.text',
  transform: ['firstWord', 'toUpperCase']
}
```

If `content.text` points to `'Tomorrow will be a perfect day, he told himself.'`, the result of the transform pipeline will be `'TOMORROW'`.

