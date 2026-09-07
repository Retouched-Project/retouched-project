# HTTP Discovery

Before connecting to the registry server via TCP, the controller app resolves the server address through an HTTP-based discovery mechanism.

## Mechanism

The controller makes an HTTP GET request to a well-known URL. The response is a JSON array of registry server addresses (as strings). The controller shuffles this list and attempts to connect to each address in order until a TCP connection succeeds.

The shuffle is what spreads controllers across the servers on offer, so a client that walked the list in order would send everyone to the first one.

## Response Format

```json
["registry1.example.com", "registry2.example.com"]
```

Each string is a host address on its own, with no port. The controller supplies the port itself, which is `8088` in every SDK.

## Fallback

If the HTTP request fails, whether from a network error or unparseable JSON, the controller falls back to a default registry host it carries in its own configuration. The discovery list only ever replaces that default; it is not required for the controller to reach a registry at all.

## Notes

- This mechanism is only used by the controller app, not by game hosts. Game hosts are typically configured with a fixed registry address.
- The original Brass Monkey registry servers are no longer operational. For Retouched, the registry server runs locally or at a user-specified address.