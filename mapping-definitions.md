# Mapping definitions

To specify how data from a service is mapped and transformed to a schema – and how to map and transform it back to the service – you set up a mapping definition, potentially with transform and mutate functions.

This is the full mapping definition:

```javascript
{
  id: <string>,
  schema: <schema id | array of schema ids>,
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
  mutate: <mutators pipeline>,
  filterFrom: <filters pipeline>,
  filterTo: <filters pipeline>
}
```

