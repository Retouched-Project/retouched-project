# HTTP Discovery

Before connecting to the registry server via TCP, the controller app resolves the server address through an HTTP-based discovery mechanism.

## Mechanism

The controller makes an HTTP GET request to a well-known URL. The response is a JSON array of registry server addresses (as strings). The controller shuffles this list and attempts to connect to each address in order until a TCP connection succeeds.

## Response Format

```json
["registry1.example.com:8124", "registry2.example.com:8124"]
```

The response is a JSON array of strings. Each string is a host address that the controller can attempt to connect to.

## Fallback

If the HTTP request fails (network error or JSON parse error), the controller falls back to a default or cached registry address.

## Notes

- This mechanism is only used by the controller app, not by game hosts. Game hosts are typically configured with a fixed registry address.
- The original Brass Monkey registry servers are no longer operational. For Retouched, the registry server runs locally or at a user-specified address.