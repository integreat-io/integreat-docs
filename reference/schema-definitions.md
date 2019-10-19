# Schema definitions

Schemas are common data shapes, that data from services are mapped to, and that data going to a service is mapped from. It is the very reason that Integreat may transfer data between different services without a common format.

A typical schema may look like this:

```text
{
  id: <string>,
  service: <serviceId>,
  shape: <shape>,
  access: <auth def>
}
```

