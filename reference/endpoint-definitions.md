# Endpoint definitions

Endpoints – in Integreat's terms – are different ways of calling a service API. For a REST API, the service typically has a base url, and the different paths to the available resources are defined as endpoints. For a SOAP API, there's one url, but the different operations may be defined as endpoints. For GraphQL APIs, there might be one endpoint for all query operations, and one endpoint for each mutation operation. These are just quick examples – see the Recipes section for more.

Each service must have at least one endpoint defined, but there's no upper limit to how many you may set up.

Endpoint definitions are included in the `endpoints` array on a [service definition](service-definitions.md), and will typically look like this, although some alternatives are mentioned below:

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
    path: <path string>,
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

The principles of endpoint mapping are much the same way as for schema mappings, with everything defined as going _from_ the service _to_ Integreat.

#### Endpoint paths

First of all, there is a `path` property, which is dot notation string that points to a property in the data from the service. When this is set, it is used to extract a part of the data to use, before passing it on to schema mapping. This is especially handy when several endpoints retrieves the same type of data, but wrapped in different object structures. The `path` property is also used when sending data to a service. After the data has been mapped with schema mappings, it is set on an empty object according to the `path` property.

In fact, the `path` property is so commonly used that you may simple set the path string directly on the `mapping` property, when the path is all you need. `mapping: 'content.articles'` is the exact same as `mapping: { path: 'content.articles' }`.

But what if it doesn't make sense to have the same path for retrieving and sending data? Instead of `path` you may use another pair of properties: `requestPath` and `responsePath`. These are just like `path`, but used for the request \(data being sent to the service\) and the response  \(data coming from the service\).

#### Request object mapping

The three path properties will cover most cases, but if you need more control over the data being sent to the service, there's always the `requestBody` object. It lets you define the shape of the request body, much the same way you define attributes and relationships in schema mappings. The keys of the `requestBody` object are dot path notations for properties on the request body, that are typically set to path strings for the `request` object. This means that you can build a request body from anything that is available on a [request object](../advanced-topics/writing-adapters/request-objects.md), like `params` or the prepared `endpoint` options. 

Imagine that you have an action with a `section` param, that you would like to send to the service together with the actual `data`. Your endpoint definition would look something like this:

```javascript
{
  match: {...},
  mapping: {
    path: 'data',
    requestObject: {
      'meta.section': 'params.section',
      'content': 'data'
    }
  },
  options: {...}
}
```

With the action payload `{ section: 'news', data: [ { id: 'article1', type: 'article' }, { id: 'article2', type: 'article' } ] }`, this would result in the following body being sent to the service:

```javascript
{
  meta: {
    section: 'news'
  },
  content: [
    { id: 'article1', type: 'article' },
    { id: 'article2', type: 'article' }
  ]
}
```

{% hint style="info" %}
Note that in this example we're also imagining that we have schema mappings that doesn't change the data. If we had mappings that change the data items, this would affect the data in the `content` array in the example.
{% endhint %}

### `options`

The content of the `options` object is entirely up to the adapter used for a service. Integreat puts no requirements on what it may hold, and it is passed as-is to the service's `prepareEndpoint()` method on setup, together with the `options` object on the service definition. When an endpoint is picked for a request, Integreat will provide the value return by `prepareEndpoint()` for that endpoint in the `endpoint` property on the `request` object.

An `options` object will often hold a `uri`, maybe some form of `method` property. You should check the adapter documentation for specifics, however.

