---
description: How to write an authenticator.
---

# Authenticator interface

An authenticator is an object with a specified set of methods:

```javascript
{
    authenticate: async (options) => authentication,
    isAuthenticated: (authentication) => boolean,
    asObject: (authentication) => ({ ... })
}
```

This object may be extended with other `as...()` methods to support special needs for different protocols or families of adapters, like `asHttpHeaders()`, which may be used by any adapter communicating over HTTP. See the description below.

## The authenticator methods

The methods are listed in logical rather than alphabetic order.

### `authenticate`

```javascript
authenticate: async (options) => authentication
```

Based on the given `options` object, the `authenticate()` method will try to authenticate with the service. This may be through a simple HTTP request, several calls to different APIs in sequence, or something completely different. Every implementation detail is up to the authenticator as long as it returns in due time with a status.

The `options` object is the one defined on the `auth` object on the service definition. It is completely up to the authenticator to decide what this object should hold, but keep in mind that the end user will have to set its properties, so keep an eye to what other authenticators are doing and try to stay close to any norm when appropriate.

If a connection is opened as part of the authentication call, it should be closed before returning, as there will be no way for Integreat to close it again otherwise.

The returned `status` may be one of four string values:

* `accepted` – the authentication were successful
* `rejected` – the authentication call went through, but with a negative response
* `timeout` – the call timed out and may be retried
* `error` – some other error occurred

The method should always run the authentiation, even when the internal state indicates that it has already ran, as e.g. an authentication token may have timed out, and the method is called to refresh it.

### `isAuthenticated`

```javascript
isAuthenticated: (authentication) => boolean
```

This method will simply return `true` when the internal state indicates that authentication has already been run and was successful. If not, it returns `false`. This should be done without any external calls, and it is therefore not expected to guaranty that the authentication is still valid, though it should do the best with what it has.

If for instance an authentication token has been retrieved, but is timed out on the server, `isAuthenticated()` should still return `true`, as it should not check with the server. If, on the other hand, the token came with an expiration time, the method should check whether this time is passed before returning `true` or `false`.

### `asObject`

```javascript
asObject: (authentication) => ({ ... })
```

When authenticated, this method will return an object with the properties relevant for this authenticator. It depends completely on the adapter whether this makes sense or not.

As an example, the simplest authenticator is the `token` authenticator, which will accept an object with some properties and returns the same object when `asObject()` is called. An adapter may specify in its documentation that this is the authenticator it expects, and what props to set on the `options` object. For other adapters, this authenticator will not help at all.

Another authenticator may call an external service to retrieve a token, and when `asObject()` is called, it returns the object `{token: "x8q17fr3i0"}`, as an random example. Depending on the adapter, this may or may not make sense.

Other `as...()` methods may be more useful in some cases. Read on …

### `as...`

```javascript
as...: (authentication) => ({ ... })
```

As mentioned, the object returned by the authenticator may have any number of addiational `as...()` methods. They all follow the same logic as `asObject()`, but their return object may be quite different. They may not even return an object – any type that makes sense to the target adapters will do.

The most common is `asHttpHeaders()`, which will transform the state of the authenticator to a set of HTTP headers. For instance, the `asHttpHeaders()` method on the `oauth2` authenticator will return an object like `{Authorization: "Bearer x8q17fr3i0"}`.

For now, it is up to the end user to pick the appropriate authenticator, however, so adapter writers should document what they expect from an authenticator, preferably with examples of authenticators that are known to deliver on these expectations. An adapter may expect an `as...()` method, but should fail with an error response or try without authentication, if the given authenticator does not support it.

{% hint style="info" %}
In the future, Integreat may specify a way for adapters to programmatically describe what kind of authenticators they need, so that Integreat might validate whether an authenticator will work or not.
{% endhint %}

