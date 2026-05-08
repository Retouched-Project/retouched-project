# onHostConnected

A push notification sent by the registry server when a new game host registers.

## Push

| Field | Value |
|-------|-------|
| **Method** | `onHostConnected` |
| **Arguments** | 1 |

| # | Type | Description |
|---|------|-------------|
| 1 | `BMRegistryInfo` | The newly registered game host's identity, address, and slot info. |

## Behavior

### Controllers

The controller adds the new host to its local list. If a host with the same `slotId` already exists, it is replaced.

### Game Hosts

A game host also receives `onHostConnected`. If the incoming `BMRegistryInfo` matches its own `deviceId`, the host adopts the server-assigned [`slotId`](slots.md) from the notification and updates its visual slot display.