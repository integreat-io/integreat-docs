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
    params: {...},
    filters: {...}
  },
  requestMapping: <path string | mapping object>,
  responseMapping: <path string | mapping object>
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
* `params`: You may match specific parameters defined on the request by setting the params this endpoint may handle as keys on the `params` object. The value should be eithter `true` or `false`, to indicate the parameter is required or not. When finding a matching endpoint for a request with a certain parameter, endpoints that require this parameter are given priority over endpoint where it is optional, which again are given priority over endpoints that does not specify the parameter at all.
* `filters`: An object with paths as keys and [JSON Schema](http://json-schema.org) as values. All provided paths must be valid according to their schemas for this endpoint to be a match. Currently, this is used to map request data only, so every path should start with `data.`.

Defining a `match` object is not the easiest part of Integreat, and if you're not careful you might end up having Integreat picking other endpoints than you expect. A general rule is to have as few endpoints as possible, start with the more general, and do overrides with more specific endpoints as needed.

The power you get from this, however, is the generic action-based interface to services and data sources, where you can access their data without knowing anything about how they are defined.

### `requestMapping`

The `requestMapping` can be a path string or a mapping object, that lets you shape the data being sent with the request to a service – sometimes refered to as the request body.

Default behavior for endpoints without a `requestMapping` object is to send any mapped data as the request body. When there's no `requestMapping` object on the endpoint and no `data` in the action payload, the request will simply be sent without data.

When you supply a path string in dot notation, any mapped data will be set on this path on an empty object before being sent as the request body. With the data `[ { id: 'ent1', type: 'entry' } ]` and `requestMapping: 'content.items'`, the data sent with the request will be `{ content: { items: [ { id: 'ent1', type: 'entry' } ]`.

When you supply a mapping object, the keys of the `requestMapping` object are dot path notations for properties on the target request body. The value of these keys are most typically set to path strings pointed at values in the `request` object.

{% hint style="info" %}
We're using the same notation as for attributes and relationships on schema mappings, and for `responseMapping`. However, the `requestMapping` is modelled after what you want to send _to_ the service, instead of modelling what you would like to get `from` the service, as in the other cases. This is probably what you would expect, but note that transform pipelines keep their direction. To apply a transform function in `requestMapping`, you would have to define a `transformTo` pipeline or a `transform` pipeline with `.rev()` functions.
{% endhint %}

You can build a request body from anything that is available on a [request object](../advanced-topics/writing-adapters/request-objects.md), like `params` or the prepared `endpoint` options, or even `data`. Note that at this point we're talking about data that has been mapped with schema mappers.

Imagine that you have an action with a `section` param, that you would like to send to the service together with the `data`. Your endpoint definition would look something like this:

```javascript
{
  match: {...},
  requestMapping: {
    'meta.section': 'params.section',
    'content': 'data'
  },
  options: {...}
}
```

With the action payload ...

```javascript
{
  section: 'news',
  data: [
    { id: 'article1', type: 'article' },
    { id: 'article2', type: 'article' }
  ]
}
```

... this would result in the following body being sent to the service:

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

### `responseMapping`

The `responseMapping` is the opposite of `requestMapping`, and is used to extract a part of the data coming from the service, before passing it on to schema mapping. This is especially handy when several endpoints retrieves the same type of data, but wrapped in different object structures, or when the status of the response is set in the data in way not uniform to the adapter.

As for `requestMapping`, the `responseMapping` may be a path string in dot notation. In this case, the path is extracted from the response data, and this is returned as data in the response.

The `responseMapping` may also be a mapping object, with the following allowed keys:

* `data`: The data mapped to this key will be returned as response data
* `status`: Should map to a string value with any of the allowed status values, and will be set as the status of response, e.g. `'ok'` to signal a successful response.
* `error`: An error message for the response, mapped from the data. Will be disregarded when status is `'ok'`.

The value of each of these keys may be a path string or an object with `path`, `transform`, etc. just as for attributes and relationships on schema mappings.

### `options`

The content of the `options` object is entirely up to the adapter used for a service. Integreat puts no requirements on what it may hold, and it is passed as-is to the service's `prepareEndpoint()` method on setup, together with the `options` object on the service definition. When an endpoint is picked for a request, Integreat will provide the value return by `prepareEndpoint()` for that endpoint in the `endpoint` property on the `request` object.

An `options` object will often hold a `uri`, maybe some form of `method` property. You should check the adapter documentation for specifics, however.

