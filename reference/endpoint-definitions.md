# Endpoint definitions

Endpoints – in Integreat's terms – are different ways of calling a service API. For a REST API, the service typically has a base url, and the different paths to the available resources are defined as endpoints. For a SOAP API, there's one url, but the different operations may be defined as endpoints. For GraphQL APIs, there might be one endpoint for all query operations, and one endpoint for each mutation operation. These are just quick examples – see the Recipes section for more.

Each service must have at least one endpoint defined, but there's no upper limit to how many you may set up.

Endpoint definitions are included in the `endpoints` array on a [service definition](service-definitions.md), and looks like this:

```javascript
{
  id: <string>,
  match: {
    schema: <schema id | array of schema ids>,
    scope: <'collection'|'member'|'all'>,
    request: <request type>,
    params: {...}
  },
  mapping: {
    requestPath: <path string>,
    responsePath: <path string>,
    requestBody: {...}
  },
  options: {...}
}
```

## Endpoint definition properties

### `id`

Endpoints could have an `id`, but it's not mandatory. When it's present, it should be unique among all endpoints for that service. The only practical use for it is to override the `match` criterias and ask for a specific endpoint instead of letting Integreat picking the closest fit for a request.

See Dispatching actions for how to ask for an endpoint by `id`.

### `match`

This object defines a set of criterias that tell Integreat when an endpoint might be a good fit for a request. It is possible to skip the `match` object entirely, which will make this a fall back endpoint, that is used when no other endpoint has a match. If you have several endpoints without a match object, the first one in the `endpoints` array will be used. For services with only one endpoint, no `match` object is needed, unless you want to limit what types of requests might be sent to the service.

The following criterias are available:

* `schema`: By specifying a schema `id`, only requests for data of this type will match this endpoint. To match several schemas, use an array of ids.
* `scope`: Set to `member` to match only requests for one data item \(actions with an `id` on the payload\), and `collection` to match only requests for sets of data items \(actions without an `id`\). Not specifying scope will match both `member` and `collection`, which is the same as setting it to `all`. 
* `request`: This matches a request type, which may be `QUERY`, `MUTATION`, or `REMOVAL`. This way you can have one endpoint for querying data and another for mutating it, while disallowing removal by not having an endpoint that matches it.
* `params`: Finally, you may match specific parameters defined on the request by setting the params this endpoint may handle as keys on the `params` object. The value should be eithter `true` or `false`, to indicate the parameter is required or not. When finding a matching endpoint for a request with a certain parameter, endpoints that require this parameter are given priority over endpoint where it is optional, which again are given priority over endpoints that does not specify the parameter at all.

Defining a `match` object is not the easiest part of Integreat, and if you're not careful you might end up having Integreat picking other endpoints than you expect. A general rule is to have as few endpoints as possible, start with the more general, and do overrides with more specific endpoints as needed.

The power you get from this, however, is the generic action-based interface to services and data sources, where you can access their data without knowing anything about how they are defined.

### `mapping`

The `mapping` object lets you specify mapping for the data coming from the service and data being sent to the service – for this endpoint. This works together with the schema mappings. Data coming from a service is first mapped with the endpoint mapping, and the result of this mapping is passed on to the relevant schema mappers. When sending data to a service, it will first be mapped with schema mappers, before it is mapped with the endpoint mapping.

Default behavior for endpoints without a `mapping` object is to leave everything to the schema mappers. The response body from the service is passed directly to the schema mappers. For requests to a service, any mapped data is sent as the request body. When there's no `mapping` object on the endpoint and no `data` in the action payload, the request will simply not have a `body`.

The principles of endpoint mapping are much the same way as for schema mappings, with everything defined as going _from_ the service _to_ Integreat. The property names are a bit different, though:

* `path`: This is a dot path notation on the data object coming from or going to the service, and is used to extract a part of the data that will be used for futher mapping when data is coming from the service, and to set the data going to the service.
* `pathTo`: This is a dot path notation for Integreat's request/response object, and is seldom used.
* `fields`: This is an object where the keys refer to properties on the `request` and/or `response` object and the value of these keys are string paths in dot notation on the body object. You may also specify a field mapping object with the string paths for the body object as a `path` property and optional `default` and `defaultRev` values. The former is used when a value is missing on the `response` object and the latter when a value is missing on the `request` object.

Note that the direction of this mapping is _from_ the request object _to_ the service, which is the oposite of how schema mappings are defined. The reason for this is that we're now talking about requests – _to_ the service, while schema mappings are defined from the point of view of data responses – _from_ a service.

Hopefully, this will all be so much clearer with an example:

```javascript

```

### `options`

The content of the `options` object is entirely up to the adapter used for a service. Integreat puts no requirements on what it may hold, and it is passed as-is to the service's `prepareEndpoint()` method on setup, together with the `options` object on the service definition. When an endpoint is picked for a request, Integreat will provide the value return by `prepareEndpoint()` for that endpoint in the `endpoint` property on the `request` object.

An `options` object will often hold a `uri`, maybe some form of `method` property. You should check the adapter documentation for specifics, however.

