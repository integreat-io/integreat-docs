# Mapping definitions

To specify how data from a service is mapped and transformed to a schema – and how to map and transform it back to the service – you set up a mapping definition, potentially with transform and mutate functions.

Note that mappings will be used to map data both from and to a service, and in the description below, we'll explain how this happens.

This is the full mapping definition:

```javascript
{
  id: <string>,
  path: <path string>,
  attributes: {
    <attribute id>: {
      path: <path string>,
      transform: <transform pipeline>,
      transformTo: <transform pipeline>
    },
    ...
  },
  relationships: {
    <relationship id>: {
      path: <path string>,
      transform: <transform pipeline>,
      transformTo: <transform pipeline>
    },
    ...
  },
  mutate: <mutate pipeline>,
  mutateTo: <mutate pipeline>,
  filter: <filter pipeline>,
  filterTo: <filter pipeline>,
  qualifier: <path string>
}
```

## Mapping defintion properties

### `id`

Optional unique `id` for a mapping definition. This is the `id` referenced from a service definition, but is not needed when a mapping is defined directly on the service.

### `path`

A path into the service data, in Integreat's extended dot notation. The path on the top level of a mapping should point to where the data for each item is, which will typically be an array – at least when the data contains a collection. For example `data.items[]`.

Attributes and relationships have their own `path` properties, which will start from the point the mapping `path` points to.

When mapping data _from_ a service, the `path` will be used to extract from this data, and everything "above" the extracted data will be dropped. When going _to_ the service, the data mapped from the schema wil be set at the specified `path` on an empty object.

See the Path format for more on how to specify a path.

### `attributes` and `relationships`

The fields of a schema is separated into `attributes` and `relationships`, and this distinction is used in mappings too, even though they are defined in the exact same way. Put simply, when a field references data items with `id` and `type` \(schema\) it's a relationship – anything else is an attribute. When mapping to a relationship, the value mapped value will be the id of the relationship. For attributes, the value is the value.

Both `attributes` and `relationships` may be defined in a short form, with a path string instead of a field definition object. `title: 'segment.headline'` is a shorter form of `title: {path: 'segments.headline'}`. This is handy when you don't need to define a `transform` pipeline.

#### `path`

The path property on `attributes` and `relationships` points to the actual value to be used for the field. If a `transform` pipeline is specified, the value will be sent through the pipeline before it is cased to the type specified in the schema.

The `path` is relative to the `path` defined on the mapping itself, and will be treated in the same way: When going _from_ the service, the path is used to extract a value from the data. When going _to_ the service, the transformed value is set at the specified path on the data being sent to the service.

#### `transform`

A `transform` pipeline defines one or more transformator functions that the mapped value will be passed through, from left to right, before being casted with the relevant type according to the schema – when mapping _from_ a service.

A transformer is basically a function that will accept a value and return a value, but may do anything with the value in between. Some transformers may be an object with a `from` function and a `to` function – so-called two-way transformers. In these cases the `from` function is used when mapping _from_ a service, and the `to` function is used when going \(you guessed it\) _to_ the service. A one-way transformer \(a simple function\) will only be used _from_ a service, and will be skipped when mapping _to_ it.

Note that when transforming _to_ a service, the transformers will be applied in the opposite direction – from right to left.

If you, as an example, have two one-way transformators, one that turns a string to uppercase and another that returns only the first word in a string, you might specify an attribute like this:

```javascript
bigWord: {
  path: 'content.text',
  transform: ['firstWord', 'toUpperCase']
}
```

If `content.text` points to `'Tomorrow will be a perfect day, he told himself.'`, the result of the transform pipeline will be `'TOMORROW'`. When mapping this back to the service, this will result in `{content: {text: 'TOMORROW'}}`, as one-way transformers are only used _from_ a service.

In another case, you may have a two-way transformer called `ageToYear`, that will take an age and return the corresponding birthyear going _from_ the service, and do the opposite going back. When the value `42` is mapped _from_ the service data, `transform: ['ageToYear']` will transform it to `1976`. When sending the data back _to_ the service, this same transform pipeline will know to return `42`.

Note that transformators should prefarably be pure functions. Only when it is absolutely necessary, should a transformator rely on external data, like the current year from the system running Integreat, and it should never have side-effects.

See Writing transformators for more details.

#### `transformTo`

The `transformTo` property takes a transformator pipeline, just as `transform`, but will only be used for transforming a value _to_ a service. It should be used when you need to define a different pipeline for transforming field values _to_ a service. The transformattors will be applied from right to left.

When `transformTo` is not specified, the `to` function on two-way transformers on the transform pipeline will be used.

### `mutate`

A `mutate` pipeline is much like the transformation pipelines, but instead of working on individual fileds, it may change the entire data object. A mutator is a function that accepts an object in Integreat's data format, and returns an object in Integreat's data format, but may do anything with the data as long as it doesn't break the format.

Going _from_ a service, Integreat will first map and transform the individual fields, cast the object to the correct schema, and then run it throught the mutation pipeline, applying matations from left to right.

When mapping back _to_ a service, any mutations will be applied to the casted object – before transforming and mapping fields. This is done by applying any `to` functions the mutators might have, from right to left.

### `mutateTo`

As an alternative to using the `mutate` pipeline with `to` functions, you may specify a separate pipeline for mutating _to_ a service. When a `mutateTo` pipeline is defined, the `mutate` pipeline will only be used for mutating from the service.

`mutateTo` is applied exactly as the `mutate` pipeline – from right to left, after casting, before transforming and mapping fields.

### `filter`

The last stop for data coming from a service, is the filter pipeline, which consists of functions that – given the data object – may filter away objects by returning `false`. To be accepted into Integreat, a data object needs to get `true` from every filter function in the `filter` pipeline. Functions are applied from left to right. This has no logical consequence, as the result is ANDed, but as the filtering stops with the first `false`, any filters that are computational heavy should be placed towards the right.

When no `filter` pipeline is specified, alle objects are accepted.

Although it is not very common, filters may be object with `from` and `to` functions, and objects will be run through the pipeline with any `to` functions, from right to left, when going _to_ a service.

Filtering _from_ a service happens after the data has been mapped, transformed, mutated, and casted, making sure that a filter function will always operate on Integreat's standard data format. Going _to_ a service, the data object is casted before the filter pipeline is applied, and the mutation, transforming, and mapping will only happen if the object passes all the filters.

### `filterTo`

This works as the `filter` pipeline, but  is used when going _to_ a service. A data object is casted before the `filterTo` pipeline is applied, but filtering happens before any mutation, transforming, and mapping.

When a `filterTo` pipeline is specified, any `to` functions on the `filter` pipeline will be disregarded.

### `qualifier`

When a service returns data for several schemas, Integreat needs a way to recognize which schema to use for each item in the data. For some services, the different schemas may be find on different paths in the data, so specifying different paths on each mapping is sufficient. But when all items are returned in one array, for instance, you need to specify qualifiers for the mappings.

A qualifier is simply a path with an expression that will evaluate to `true` or `false`. If a mapping has qualifiers, it will only be applied to data that satisfies all its qualifiers. Qualifiers are applied to the data at the mapping's `path`, before it is mapped and transformed.

An example of two mappings with qualifiers:

```text
...
mappings: {
  entry: {
    attributes: {...},
    qualifier: 'type="entry"'
  },
  admin: {
    attributes: {...},
    qualifier: [
      'type="account"',
      'permissions.roles[]="admin"'
    ]
  }
}
```

When a qualifier points to an array, the qualifier returns true when at least one of the items in the array satisfies the condition.

