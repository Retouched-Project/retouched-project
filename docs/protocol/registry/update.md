# registry.update / onHostUpdate

Allows a game host to push metadata changes to the registry server, which then broadcasts the update to all connected controllers.

## Request: registry.update

| Field | Value |
|-------|-------|
| **Method** | `registry.update` |
| **Return Method** | The caller's own, never a fixed name. See [Return Values](../objects/bm-invoke.md#return-values). |
| **Arguments** | 1 |

### Arguments

| # | Type | Description |
|---|------|-------------|
| 1 | `BMRegistryInfo` | The host's updated registry info. Typically only `currentPlayers` changes. |

The game host calls this method whenever its player count changes (a controller connects or disconnects). Updates are rate-limited on the client side; if multiple changes occur within ~1 second, only the latest update is sent after a short delay.

## Push: onHostUpdate

The server broadcasts the update to all connected controllers:

| # | Type | Description |
|---|------|-------------|
| 1 | `BMRegistryInfo` | The updated host info. |

Upon receiving `onHostUpdate`, the controller updates the matching host entry in its list by `slotId` and refreshes the `currentPlayers` count.