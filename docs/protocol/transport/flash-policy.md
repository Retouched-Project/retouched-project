# Flash Socket Policy

Adobe Flash Player requires a cross-domain socket policy before it will allow a `Socket` connection to a non-standard port. Since a Flash game connects out to a controller, the **controller** is the side that has to serve the policy, and it does so inline on the port it already listens on.

## Request

When a Flash game connects, it may send a policy file request before any BM traffic:

```
<policy-file-request/>\0
```

This is a null-terminated ASCII string (23 bytes including the null terminator).

## Response

Upon detecting a policy request, the controller responds with:

```xml
<?xml version="1.0"?><cross-domain-policy><allow-access-from domain="*" to-ports="1008-49151" /></cross-domain-policy>\0
```

The response is also null-terminated. After sending it, the controller closes the connection, and the Flash game opens a fresh one for actual BM traffic.

## Detection

The TCP listener compares the first 16 bytes of each new connection against the start of `<policy-file-request/>`. If they match, the connection is treated as a policy request rather than a BM handshake.

Matching on a prefix rather than the whole string means the trailing null is never part of the test, so a client that omits it is still recognised. Sixteen bytes is also short enough to decide before the rest of the request has necessarily arrived.

This check happens on the same port as the normal TCP listener. No separate policy server port is required, though a dedicated `PolicyServer` can also run on a separate port if needed.

## Port Range

The policy grants access to ports 1008-49151, covering the full range of non-privileged ports typically used by BM.