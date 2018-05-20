---
description: >-
  Integreat is a translator between services and datasources, built in
  JavaScript. It's a gateway API, an integration layer, or an adapter for your
  APIs, depending on how you use it.
---

# Introduction

{% hint style="info" %}
This documentation is very much work in progress and is written for the upcomming 0.7 release of Integreat. For information on the 0.6 version, see [the README on Gihub](https://github.com/integreat-io/integreat).
{% endhint %}

The basic idea of Integreat is to make it easy to define a set of data sources and expose them through a well defined interface, to abstract away the specifics of each source, and map their data to defined datatypes.

This is done through:

* adapters, that does all the hard work of communicating with the different sources
* a definition format, for setting up each source with the right adapter and parameters
* a `dispatch()` function that sends actions to the right adapters via internal action handlers

It is possible to set up Integreat to treat one source as a store or a buffer for other sources, and schedule syncs between this store and the other sources.

Finally, there will be different interface modules available, that will plug into the `dispatch()` function and offer other ways of reaching data from the sources – such as out of the box REST or GraphQL APIs.

