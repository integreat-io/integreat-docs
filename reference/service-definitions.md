# Service definitions

Service definitions are at the core of Integreat, as they describe how to connect to each service, how to fetch data and map it to the relevant schemas, and how to map from schemas and send data back to the service.

The service definitions object looks like this:

```javascript
{
  <id>: {
    transport: <transport id>,
    options: { ... },
    auth: <auth def | auth id | true>,
    meta: <schema id>,
    endpoints: [
      <endpoint definition>,
      ...
    ],
    mappings: {
      <schema id>: <mapping definition | mapping id>,
      ...
    }
  },
  ...
}
```

## Service definition properties

### `id`

This is a unique id for a service definition, which will be used to reference the service throughout Integreat. A typical use for the `id` is when you want to specify the service in an action or in a flow.

### `transport`

Specifies the transport to use for this service. This is an `id` string, that should match one of the transports passed to the `Integreat.create()` method.

### `options`

The options object is passed blindly to the transport, so its content will vary based on the transport you choose. It is first passed to the `prepareEndpoint()` method on the adapter together with each endpoint's `options` object, and then to the `connect()` method on the transport.

The most common options property is `baseUri`.

### `auth`

The `auth` property could either be an auth definition or the `id` to an auth definition object, if the service requires authentication. This tells Integreat how to acquire the necessary authentication token or whatever the service needs.

In cases where the service is authenticated by other means, e.g. by including username and password in the uri, set the `auth`property to `true` to signal that this is an authenticated service and trigger Integreat's security measures.

### `meta`

&lt;depricated&gt;

### `endpoints`

A service should be defined with at least one endpoint, but there's no upper limit to how many endpoints you may supply. You may think of endpoints as overrides to the service definitions. Each endpoint may have its own transport options, schema mappings, etc., that will override or merge with what is defined on the service.

An endpoint is defined with a `match` object, specifying the cases the endpoint is intended to serve, with varying degree of specificity. This is powerful, as it lets Integreat respond to generic requests for data and match it to the relevant endpoint for each specific service, without requiring the "requester" to know anything about the service it is getting data from. It is also potentially quite tricky, as one mistake in the `match` objects could mess up the matching algorithm.

Integreat will always pick the endpoint with the highest specificity that matches the request in question. A typical approach is therefore to define one or more general endpoints, and set up "exceptions" with more specific matching criteria.

An endpoint may also have an `id`, which allows an action to be defined with a specific endpoint in mind, and will override the `match` objects. This should only be used as a last resort, though, as it is always best to keep service specifics away from action definitions.

The order of the endpoints in a service definition has no effect, except for cases where two endpoints with the same level of match specificity would both match a request. In this case, the first match will be picked.

See Endpoint definitions for details.

### `mappings`

A mapping defintion tells Integreat how to map between a schema and the data coming from a service or being sent to a service. You should include one mapping definition for every schema a service can handle. Note that endpoints may have their own set of schema mappings, that will override the service schema mappings.

The `matching` object should have the `id` of schemas as keys, with the `id` of a mapping definition or a complete mapping definition object, as a value. The former option is for reusing mapping definitions across services and schemas, or for cases when it is easier to have mapping definition in separate files – e.g. when service definitions grow big.

See Mapping definitions for details.

