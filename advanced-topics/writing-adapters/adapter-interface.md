---
description: A run-trough of the methods needed on an adapter.
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
    send: async (request, connection) => response,
    normalize: async (response, request) => response,
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

The purpose of `serialize()`, is to modify the `data` structure of the request to whatever the adapter will send to the service. Anyting that makes sense for the adapter is allowed, including keeping the data untouched.

This could also be done in the `send()` method, but there are two reasons to implement this as a seperate method. First of all, it is good to have methods with limited responsibilities. It will make the adapter code clearer to understand. Secondly, this allows optimizations in Integreat in the future, e.g. if there's a need for a pattern where the same data is sent to a service several times, this data could still be serialized only once.

It should go without saying, that the minimal implementation is to simply return the `request` object.

### `send`

```javascript
send: async (request, connection) => response
```

Here comes the working horse. The `send()` method will receive the `request` object that has been passed through the `serialize()` method, and use it to send the relevant request to the service. How this is done is completely up to the adapter, and will most likely be very different from adapter to adapter. That is kind of the purpose …

The `connection` object is the one that was returned from the `connect()` method.

When completed, the method must return a response object with `status: "ok"` and any data returned from the service, if the request was a success. This will be passed to the `normalize()` method before Integreat starts using it, so any raw format is okay, as long as `normalize()` does its job of tranforming it to a structure of JavaScript basic types.

There are also available statuses for signaling errors, but a `response` object should be returned in any case.

See the documentation on [request objects](request-objects.md#the-request-object) and [response objects](response-objects.md#the-response-object) for more details on what to expect from Integreat and what Integreat expects in return.

### `normalize`

```javascript
normalize: async (response, request) => response
```

This is the counterpart to the `serialize()` method, and will transform the `data` returned in the `response` object from `send()` into a structure of plain JavaScript types.

The `request` object is provided only for reference, in case any of its properties is needed to transform the data correctly. To be clear, it is given the `request` object returned from its own `serialize()` method. One use case is the `path` property frequently seen in the endpoint `options`, that specifies a subset of the data to be returned. This may also be done through normal Integreat mapping, but doing this in `normalize()` might make it more efficient, as it could drop part of the data set right away.

Just as for `serialize()`, this normalization could – and can – happen in the `send()` method, but keeping it in its own method is good for seperation of concerns, i.e. cleaner code, and it allows Integreat to do optimizations in the future.

The minimal implementation for `normalize()` would be, yes, you guessed it, just to return the `response` object.

### `disconnect`

```javascript
disconnect: async (connection) => {}
```

Given the `connection` object, this method should do whatever is necessary to close the connection. Integreat does not give any guaranty on when this will happen. It will usually keep the connection open after completing a request, but it might decide to close it after an idle time and during a clean shut down of the system.

After calling `disconnect()`, Integreat will not reuse the `connection` object.

The minimal implementation of `disconnect()` is to do nothing, and in any adapter that does not need a connection to its service, this is exactly what you'll do.

