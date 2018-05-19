---
description: >-
  On a higher level, an adapter will be given a request object and will respond
  with a response object, but the different methods play different parts in this
  process.
---

# Request objects

The methods that will come in contact with the request object are:

* `serialize: async (request) => request` is given a chance to serialize the data on the request before it is sent to the service
* `send: async (request) => response` does the actual sending for the request, and is expected to return a response object with the status of the request and any returned data.

A request object may look like this:

```javascript
{
    type: "QUERY",
    params: {
        type: "entry",
        id: "ent1"
    },
    data: [ ... ],
    endpoint: {
        uri: "https://url.to/source"
    },
    access: {
        ident: {id: "johnf"}
    },
    auth: { ... },
    meta: {
        typePlural: "entries"
    }
}
```

## The request object

### `data`

This may be any JavaScript structure built from basic JavaScript types.

When the `request` object is passed to `serialize()`, this data may be modified as a preparation for the `send()` method, though it may very well leave it untouched. For instance, an object structure may be serialized as a XML string, so the same request `data` may be an object at one point, and a string at a later point, and this is just as expected. The reverse process will happen for the response `data`.

### `endpoint`

The structure of the `endpoint` object is completely up to the adapter. It is regarded as adapter internals.

Before an `endpoint` object is provided with a request, this will have happened:

1. A `service` definition has a list of `endpoints`, and each of one will have an `options` object.
2.  On setup, each `options` object is sent to the adapter's `prepareEndpoint()` method together with `serviceOptions`.
3. When an action is dispatched, the most relevant endpoint is found based in the `match` criterias, and the corresponding prepared `options` object from step 1 is set on the request as the `endpoint` object.

So the `endpoint` object is made up of the `options` object on the relevant endpoint, that may have been altered and prepared by `prepareEndpoint()`. For adapters communicating over HTTP, the object will typically have an `uri` and a `method` property, while a database adapter may have a `databaseUri`, a `database` name, and a `query` object.

### `params`

This is an object with the parameters specific to this request. Typically, this will originate from the dispatched action that ended in this request being sent to the adapter. It is up to the adapter how to handle `params`. Some adapters may have properties on the `endpoint` object with placeholders for the parameters, and will do a find and replace. Others may send the params directly to the service, while the matched `endpoint` will be enought for others.

### `type`

A request `type` may currently be on of three values:

* `QUERY`
* `MUTATION`
* `REMOVAL`

The `type` is provided as information only, and it is up to the adapter to handle them appropriately. For some adapters, the information on the endpoint object may be enough to take the correct action, while others may rely more heavily on the request `type`.



