# Flash Socket Policy

Adobe Flash Player requires a cross-domain socket policy before it will allow a `Socket` connection to a non-standard port. The BM game host handles this inline on its TCP listening port.

## Request

When a Flash client connects, it may send a policy file request before any BM traffic:

```
<policy-file-request/>\0
```

This is a null-terminated ASCII string (23 bytes including the null terminator).

## Response

Upon detecting a policy request, the server responds with:

```xml
<?xml version="1.0"?><cross-domain-policy><allow-access-from domain="*" to-ports="1008-49151" /></cross-domain-policy>\0
```

The response is also null-terminated. After sending the response, the server closes the connection. The Flash client will then open a new connection for actual BM traffic.

## Detection

The TCP listener checks the first bytes of each new connection against the `<policy-file-request/>` prefix. If the incoming data matches, the connection is treated as a policy request rather than a BM handshake.

This check happens on the same port as the normal TCP listener. No separate policy server port is required, though a dedicated `PolicyServer` can also run on a separate port if needed.

## Port Range

The policy grants access to ports 1008-49151, covering the full range of non-privileged ports typically used by BM.