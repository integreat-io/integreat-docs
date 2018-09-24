---
description: An overview of the request object format.
---

# Request objects

An adapter will be given a request object and will return a [response object](response-objects.md). Different methods have different responsibilities in this process, however.

The methods that will come in contact with the request object are:

* `serialize: async (request) => request` is given a chance to serialize the data on the request before it is sent to the service
* `send: async (request, connection) => response` does the actual sending for the request, and is expected to return a response object with the status of the request and any returned data.
* `normalize: async (response, request) => response` is given the request object as a reference, in case it needs any of its properties when normalizing the response object.

A request object may look like this:

```javascript
{
    method: "QUERY",
    params: {
        type: "entry",
        id: "ent1"
    },
    data: [ ... ],
    endpoint: {
        uri: "https://url.to/source"
    },
    access: {
        status: "granted",
        ident: {id: "johnf"},
        scheme: "data"
    },
    auth: { ... },
    meta: {
        typePlural: "entries"
    }
}
```

## The request object

### `access`

Integreat authorizes a request, both its data and the request itself, against rules set on the defined schemas. A request is never sent to an adapter without passing Integreat's checks. The result of this check is an object with a `status`, e.g. the result of the check, the last `scheme` used to arrive at this status, typically `data` when it arrives with the adapter, and the `ident` used for the authorization.

The `ident` might be the most interesting for an adapter, as it provides some information on the "user" that initiated the request. The `id` might be the most interesting here, as it will often hold something close to a user name, that may be used when calling the service, when appropriate. For requests triggered by Integreat, there will be no `id`, however. Instead a `root` property will be set to `true`.

{% hint style="warning" %}
Further down the road, Integreat may come with identity mapping, where the internal `ident` may not be mapped to a identity id defined by the service configuration. It might be a good idea not to rely too much on the `ident` for now.
{% endhint %}

### `auth`

While `access` is used for authorizing requests within Integreat, `auth` is used for authorization with the external service. The `auth` object comes from the authenticator set up in the service definition, and the presice content of the object is determined by the `authentication` method set on the adapter, which refers to an actual method on the authenticator.

Not every authenticator may be compatable with your adapter, though, and Integreat will check that the selected authenticator supports the authentication method expected by the adapter. If not, Integreat will return a response with `status: "autherror"` right away.

It is good practice to recommend one or more authenticators that may be used with an adapter, in the adapter documentation. Sometimes there is a one to one relationship between an adapter and an authenticator, and you may have to write your own authenticator if your service has special authentication requirements.

### `data`

This may be any JavaScript structure built from basic JavaScript types.

When the `request` object is passed to `serialize()`, this data may be modified as a preparation for the `send()` method, though it may very well leave it untouched. For instance, an object structure may be serialized as a XML string, so the same request `data` may be an object at one point, and a string at a later point, and this is just as expected. The reverse process will happen for the response `data`.

### `endpoint`

The structure of the `endpoint` object is completely up to the adapter. It is regarded as adapter internals. Keep in mind, though, that end users will specify these options when configuring a service with your adapter, so be mindful of how you define this object, and try to stay close to how other adapters do it, when appropriate.

Before an `endpoint` object is provided with a request, this will have happened:

1. A `service` definition has a list of `endpoints`, and each of one will have an `options` object.
2.  On setup, each `options` object is sent to the adapter's `prepareEndpoint()` method together with `serviceOptions`.
3. When an action is dispatched, the most relevant endpoint is found based in the `match` criterias, and the corresponding prepared `options` object from step 1 is set on the request as the `endpoint` object.

So the `endpoint` object is made up of the `options` object on the relevant endpoint, that may have been altered and prepared by `prepareEndpoint()`. For adapters communicating over HTTP, the object will typically have an `uri` and a `method` property, while a database adapter may have a `databaseUri`, a `database` name, and a `query` object.

### `meta`

Integreat will set some properties on the `meta` object that may be useful when preparing a request, typically for substituting placeholders in URIs etc. The list is currently not long, but more may come:

* `typePlural` – the plural version of the type parameter, when one is set on the request. Schema definitions are used to get from a `type` to its plural state. When no plural is defined, this is simply the `type` with an "s" appended to it. This may be practical when you for instance ask for a data from a RESTful service, where the API is designed around the same types as Integreat is configured with, and in true RESTfulness, the endpoint is the plural of the type.

### `method`

A request `method` may currently be on of three values:

* `QUERY`: A request to get data from a service
* `MUTATION`: A request to alter data in a service
* `EXTINCTION`: A request to delete data in a service

The `method` is provided as information only, and it is up to the adapter to handle them appropriately. For some adapters, the information on the endpoint object may be enough to take the correct action, while others may rely more heavily on the request `method`.

### `params`

This is an object with the parameters specific to this request. Typically, this will originate from the dispatched action that ended in this request being sent to the adapter. It is up to the adapter how to handle `params`. Some adapters may have properties on the `endpoint` object with placeholders for the parameters, and will do a find and replace. Others may send the params directly to the service, while the matched `endpoint` will be enought for others.



