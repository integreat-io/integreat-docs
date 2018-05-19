---
description: An overview of the response object format.
---

# Response objects

The response object represents the result of a [request](request-objects.md) carried out by an adapter. Two methods on an adapter return response objects:

* `send: async (request) => response` will carry out the request and return a response object.
* `normalize: async (response, request) => response` will be given the response from `send()` and may normalize its data.

An example response object:

```javascript
{
    status: "ok",
    data: [ ... ],
    access: {
        status: "granted",
        scheme: "data",
        ident: {id: "johnf"}
    }
}
```

## The response object

### access

The `access` object is the result of Integreat's authorization. When you create a `response` object in the `send()` method, you may simple pass on the `auth` object from the request. You may also alter it, when relevant. Your adapter may for example do its own authentication of a request from Integreat, and may return a `rejected` response right away for requests that don't pass.

The properties of the access object are `status`, e.g. the result of the authorization check, the last `scheme` used to arrive at this status, and the `ident` used for the authorization

Integreat does a final authorization on the response before returning it to the dispatcher, to check that the `ident` has permissions to see the `data` on the response according to the rules set on the `schema` definitions.

{% hint style="warning" %}
Note that Integreat may introduce identity mapping further down the line, to map the `ident` on the `access` object to an identity id defined on the service configuration. So don't rely too much on the content of the `ident` object for now.
{% endhint %}

### `data`

The data returned from the request. For the response object coming from the `normalize()` method, this must be a structure of only plain JavaScript types, but the `send()` method may return anything in the data, as long as the `normalize()` method knows how to deal with it.

When the response arrives in Integreat, `data` will be mapped to the internal data format according to the relevant mapping definition.

Note that the `data` property is expected for responses with the `ok` status, but should be excluded for all other statuses.

### `error`

This is a string describing what went wrong. It should only be used with error statuses, and never with the `ok` status.

### `status`

For all successful requests, the returned `status` from an adapter should be `ok`. There are several other status codes that may be used in the case of an error:

* `notfound`: Tried to access a resource/endpoint that does not exist
* `noaction`: The action did nothing. This might not be viewed as an error in all cases, but the response will have an `error` property and no `data`
* `timeout`: The attempt to perform the action timed out
* `autherror`: An authentication request failed
* `noaccess`: Authentication is required or the provided auth is not enough
* `error`: Any other error

None of the listed error statuses should have a `data` propert, and instead a description of the error should be set on the `error` property.

An `ok` response should never have an `error` property, and is expected to have a `data` property, although it may be `null` when appropriate.

{% hint style="info" %}
A response may also have a `status` of `queued`, but a queued action will never reach an adapter, and so no adapter should ever use this status.
{% endhint %}



