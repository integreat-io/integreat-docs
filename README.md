---
description: >-
  Integreat is a translator between services and data sources of different
  kinds. It's a gateway API, an integration layer, or an adapter for your APIs –
  depending on how you use it. 100 % TypeScript.
---

# Integreat

{% hint style="info" %}
This documentation is very much work in progress and is written for the 0.8 release of Integreat.
{% endhint %}

The basic idea of Integreat is to make it easy to define a set of data sources and expose them through a well defined interface, to abstract away the specifics of each service, and map their data to defined schemas.

This is done through these basic building blocks:

* **Schemas:** Define some common data shapes, that all incoming and outgoing data are mapped to and from
* **Transformers:** Transform data from one state to another, e.g. from XML to plain JavaScript objects, from a date string to a date, text manipulation, etc.
* **Mappings:** Use json paths and transformers to mutate incoming data to a common schema, and from the schema to what another service is expecting
* **Transporters:** Transfer data to and from the different services \(the most common is http\)
* **Service definitions:** Set up a service, with the right transporter, authentications, and mappings
* **Flows:** Define a sequence of actions to execute, triggered by incoming requests, data changes, or time schedules
* **`dispatch()` function:** Sends requests as actions to the right services via a router and internal action handlers

With these building blocks, it is possible to set up Integreat in different ways, e.g.:

* As a gateway API, where incoming requests are relayed to one or more services, hiding the implementation details of each service
* As a sync job, where data are retrieved from one service and sent to another, at set intervals or triggered by a hook or a change log, etc.
* As a cacher, where incoming requests fetches data from a storage service, that is being populated with data from other services on a schedule or by demand

Integreat is built with an "everything is a service" philosophy, so whether you set up a local storage or database, a mapping to a legacy, proprietary system or a state-of-the-art open API, or you build a uniform gateway API, you use the same building blocks. Defining a _GraphQL API_ as a front to your setup, is not very different from how you define the mapping to the _SOAP API_ and the _CSV files on a FPT server_ in the back.

