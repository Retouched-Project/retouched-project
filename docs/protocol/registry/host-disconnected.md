# onHostDisconnected

A push notification sent by the registry server when a game host disconnects.

## Push

| Field | Value |
|-------|-------|
| **Method** | `onHostDisconnected` |
| **Arguments** | 1 |

| # | Type | Description |
|---|------|-------------|
| 1 | `BMRegistryInfo` | The disconnected game host's identity. |

## Behavior
The controller finds the host entry with the matching `slotId` in its local list and removes it.