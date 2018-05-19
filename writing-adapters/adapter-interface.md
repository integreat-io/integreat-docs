---
description: A run-trough of the methods to implement on an adapter.
---

# Adapter interface

Adapters are the reason Integreat can do what it does, and as more adapters are written, Integreat will be more and more powerful. The tricky part, though, is to define a single interface that may be used to query and mutate data on practically any service, data source, or database. 

The guiding principles has therefore been to leave as much of the implementation details as possible open to the adapter implementations, but to be clear on what information an adapter will get from Integreat and what Integreat expects to get back. An important part of this is the lack of specification for the the `options` objects in the configuration of the endpoints and the service itself.

An adapter is basically an object with the following methods:

```javascript
{
    prepareEndpoint: (endpointOptions, serviceOptions) => endpointOptions,
    connect: async (serviceOptions, auth, connection) => connection,
    serialize: async (request) => request,
    send: async (request) => response,
    normalize: async (response) => response,
    disconnect: async (connection) => {}
}
```

The order should hint at how this is used by Integreat.

We'll go through every method in turn below, and talk about what each method is expected to do, but note that the only method that is required to actually do something, is the `send()` method. In the most basic implementation, all the others would simply return one of its arguments and leave it to `send()` to do all the heavy lifting. But we'll see why it might be a good idea to distribute the responsbilities.

## The adapter methods

This specification is ordered logically rather than alphabetically.

### `prepareEndpoint`

```javascript
prepareEndpoint: (endpointOptions, serviceOptions) => endpointOptions
```

This method is called on setup with the `options` object from each endpoint in the service definition, together with the `options` object defined for the service itself. These are the `endpointOptions` and the `serviceOptions`.

A minimal implementation would simple return the `endpointOptions`, which would then be included as-is in any request where it is relevant, but any modification of the `endpointOptions` are permitted, as long at it makes sense within the logic of the adapter.

Some adapters might transform one or more properties to a canonical format, include default values, or transfer properties from the `serviceOptions`, to mention some examples. A good reason for doing this, is that it will only have to be done once – on setup. This may also force some operations to be moved to other methods, if they need be done per request.

Also, this is the only method that is given the `serviceOptions`, apart from `connect()`, so any relevant property should be moved to the `endpointOptions` here.

### `connect`

```javascript
connect: async (serviceOptions, auth, connection) => connection
```

For some services, it may make sense to connect to the service before sending a request. The upside of having this as a seperate method, is that Integreat will remember the `connection` object – which might be anything – between calls. An adapter should be stateless, so this is the only way to avoid connecting every time.

The `connect()` method is called before every call to the `send()` method, but the `connection` object returned from the last call is provided as the third argument. You might want to do a check that the connection has not timed out, but if everything is okay, you simply return the `connection` object. On first call, `connection` will be `null`.

The first argument is the `options` object from the service definition. It may hold anything your adapter needs. The second argument is an `auth` object, which is an instance of the authenticator that this service is set up with, given the `authOptions` for the service. This is the same `auth` object that you will receive on the request later on, and you may need it for authentication with the service.

The most basic implementation of `connect()` would just return the `connection` object it is given.

### `serialize`

```javascript
serialize: async (request) => request
```



