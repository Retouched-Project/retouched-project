# registry.list / onList

Requests the current list of registered game hosts from the registry server.

## Request

| Field | Value |
|-------|-------|
| **Method** | `registry.list` |
| **Return Method** | The caller's own, never a fixed name. See [Return Values](../objects/bm-invoke.md#return-values). |
| **Arguments** | 0 |

The controller sends this call immediately after a successful [`registry.register`](register.md). No arguments are required.

## Response: onList

The server replies by invoking whichever return method the caller named:

| # | Type | Description |
|---|------|-------------|
| 1 | `BMArray` | An array of `BMRegistryInfo` objects, one per registered game host. |

Each `BMRegistryInfo` in the array contains the host's device identity, network address, app ID, slot ID, and player counts. See [BMRegistryInfo](../objects/bm-registry-info.md).

## Behavior

### Controllers

The controller populates its host list from the received array. This is the initial discovery mechanism; subsequent changes are delivered via [`onHostConnected`](host-connected.md), [`onHostDisconnected`](host-disconnected.md), and [`onHostUpdate`](update.md) push notifications.

### Game Hosts

Game hosts can receive `onList`. They scan the array for their own `deviceId` to confirm the server-assigned [`slotId`](slots.md).