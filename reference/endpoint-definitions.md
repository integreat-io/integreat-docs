# Endpoint definitions

Endpoints – in Integreat's terms – are different ways of calling a service API. For a REST API, the service typically has a base url, and the different paths to the available resources are defined as endpoints. For a SOAP API, there's one url, but the different operations may be defined as endpoints. For GraphQL APIs, there might be one endpoint for all query operations, and one endpoint for each mutation operation. These are just quick examples – see the Recipes section for more.

An endpoint definition looks like this:

```javascript
{
  id: <string>,
  match: {
    type: <string>,
    scope: <'collection'|'member'>,
    action: <action type>,
    params: {...}
  },
  mapping: {
    fields: {...},
    path: <path string>,
    transform: [...],
    transformRev: [...]
  },
  options: {...}
}
```

