# registry.register

Registers the device with the registry server. This is the first RPC call made after the [handshake](../connection/handshake.md) completes. Both game hosts and controllers use this method, but with different `BMRegistryInfo` payloads.

## Request

| Field | Value |
|-------|-------|
| **Method** | `registry.register` |
| **Return Method** | The caller's own, never a fixed name. See [Return Values](../objects/bm-invoke.md#return-values). |
| **Arguments** | 1 or 2 (see below) |

### Arguments

| # | Type | Description |
|---|------|-------------|
| 1 | `BMRegistryInfo` | The registering device's identity, address, and app ID. See [BMRegistryInfo](../objects/bm-registry-info.md). |
| 2 | `string` | (Optional) A domain string to scope host discovery. If omitted, the device registers in the default (global) domain. |

## Host vs. Controller

| | Game Host | Controller |
|---|---|---|
| `slotId` | `1` (self-assigned) | `0` |
| `currentPlayers` | `0` (initial) | (omitted) |
| `maxPlayers` | The game's own limit | (omitted) |
| Post-registration | Starts sending pings | Calls [`registry.list`](list.md) |

`maxPlayers` has no protocol default and no protocol limit. It is whatever the game allows, bounded only by the i16 field it travels in.

Game hosts register with `slotId = 1` as a placeholder, and the server replaces it with the real one (see [Slot Allocation](slots.md)). Controllers register with `slotId = 0`, which is also what keeps `currentPlayers` and `maxPlayers` off the wire for them, since those two fields are only serialized when the slot is positive.

## Response

The server replies by invoking whichever return method the caller named:

| # | Type | Description |
|---|------|-------------|
| 1 | `boolean` | `true` if registration succeeded, `false` otherwise. |

## Post-Registration

After a successful registration:

- **Controllers** immediately call [`registry.list`](list.md) to retrieve available game hosts. The server also begins sending push notifications (`onHostConnected`, `onHostDisconnected`, `onHostUpdate`).

- **Game hosts** begin sending periodic [Ping](../messages/ping-ack.md) packets and can call [`registry.update`](update.md) to push metadata changes (e.g., player count).