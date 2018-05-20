# Core Concepts

Integreat is built on essentially three core concepts:

* **Adapters** that know how to talk to services and data sources
* **Schemas** describing the data available in a specific Integreat setup
* **Mappings** that define how to transform data from an adapter to schemas and vice versa

At the center of this is a `dispatch()` method, not unlike the one known from Redux or Flux, that will take an action and perform the relevant requests to the right services through their adapters, map data to or from the appropriate schemas, and return with a standardized response. As soon as you have setup up Integreat, the `dispatch()` method will be all you need to work with your data.

A few other concepts have important roles in most Integreat configurations:

* **Idents** represents users, roles, or actors that may have permissions to access and modify data through Integreat. These permissions are configured per schema
* **Authenticators** are a special kind of adapters that know how to authenticate with services and data sources
* **Workers** may trigger actions and perform operations across or between different services, like syncing data from one service to another
* **Pluggable APIs** sit on top of Integreat to offer access to the underlying services and data sources through well known API standards. They will know how to dispatch relevant actions from incomming requests and return the response in the format required by each API implementation

Integreat may also be set up with a **queue**, which allows the `dispatch()` method to queue some actions instead of executing them right away. A queue with a scheduler may also allow workers, which are basically higher order actions, to be dispatched at specified time intervals.

## Adapters



## Schemas



## Mappings



## Actions and the `dispatch()` method



## Supporting Concepts

### Idents



### Authenticators



### Workers



### Pluggable APIs



### Queues



