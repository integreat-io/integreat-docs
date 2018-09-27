---
description: How to write an authenticator.
---

# Authenticator interface

An authenticator is an object with a specified set of methods:

```javascript
{
    authenticate: async (options) => authentication,
    isAuthenticated: (authentication) => boolean,
    as...: (authentication) => ({ ... })
}
```

This object may be extended with as many `as...()` methods as relevant for the different protocols or families of adapters the authenticator supports. An example is the `asHttpHeaders()`, which may be used by any adapter communicating over HTTP. See the description below.

## The authenticator methods

The methods are listed in logical rather than alphabetic order.

### `authenticate`

```javascript
authenticate: async (options) => authentication
```

Based on the given `options` object, the `authenticate()` method will try to authenticate with the service. This may be through a simple HTTP request, several calls to different APIs in sequence, or something completely different. Every implementation detail is up to the authenticator as long as it returns in due time with a status.

The `options` object is the one defined on the `auth` object on the service definition. It is completely up to the authenticator to decide what this object should hold, but keep in mind that the end user will have to set its properties, so keep an eye to what other authenticators are doing and try to stay close to any norm when appropriate.

If a connection is opened as part of the authentication call, it should be closed before returning, as there will be no way for Integreat to close it again otherwise.

The returned `authentication` object will have a `status` property with one of the following string values:

* `granted` – the authentication were successful
* `refused` – the authentication call went through, but with a negative response
* `timeout` – the call timed out and may be retried
* `error` – some other error occurred

In case of the `error` status, an `error` property must be set, with a description of what went wrong.

The `authentication` object may have other properties as well, internal to the authenticator, as long as they don't interfere with the expected `status` and potiential `error` properties.

### `isAuthenticated`

```javascript
isAuthenticated: (authentication) => boolean
```

This method will simply return `true` when the given `authentication` object indicates that authentication has already been run and was successful. If not, it returns `false`. This should be done without any external calls, and it is therefore not expected to guaranty that the authentication is still valid, though it should do the best with what it has.

If for instance an authentication token has been retrieved, but is timed out on the server, `isAuthenticated()` should still return `true`, as it should not check with the server. If, on the other hand, the token came with an expiration time and this is set on the `authentication` object, the method should check whether this time is passed before returning `true` or `false`.

### `as...`

```javascript
as...: (authentication) => ({ ... })
```

The authenticator object may have any number of `as...()` methods. When the `status` of the `authentication` object is `granted`, these methods will return an object – or any other type that makes sense to the target adapters – with the properties or values relevant for each authenticator.

The simplest authenticator is the `token` authenticator, which will accept an `options` object with some properties and return the same object when `asObject()` is called – like `{token: "x8q17fr3i0"}`, as a random example. An adapter may specify in its documentation that this is the authenticator it expects, and what props to set on the `options` object. For other adapters, this authenticator will not help at all.

A common method is `asHttpHeaders()`, which will transform the `authentication` object to a set of HTTP headers. For instance, the `asHttpHeaders()` method on the `oauth2` authenticator will return an object like `{Authorization: "Bearer x8q17fr3i0"}`.

The authentication property on the adapter determines which `as...()` method will be used, and Integreat will throw an error when the authenticator specified in the service definition expects a method not found on the selected authenticator.

Adapter writers should also document what they expect from an authenticator, preferably with examples of authenticators that are known to deliver on these expectations.

