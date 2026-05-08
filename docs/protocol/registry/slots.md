# Slot Allocation

A slot ID is a numeric identifier assigned to each game host by the registry server. It is used to visually distinguish hosts and to track their lifecycle.

## How Slots are Assigned

1. The game host calls [`registry.register`](register.md) with `slotId = 1` as a placeholder.
2. The registry server assigns the actual slot ID and includes it in the `BMRegistryInfo` it broadcasts.
3. The game host receives its real slot ID via one of two push notifications:
    - [`onHostConnected`](host-connected.md): The host checks if the incoming `BMRegistryInfo` matches its own `deviceId`. If so, it adopts the `slotId` from the server's response.
    - [`onList`](list.md): Similarly, the host scans the list for its own `deviceId` and reads the assigned `slotId`.

## Slot ID Semantics

| Value | Meaning |
|-------|---------|
| `0` | Not a host (controller). No host-specific fields are serialized. See [BMRegistryInfo](../objects/bm-registry-info.md). |
| `>= 1` | Active game host. The value identifies the host's position in the registry. |

Slot IDs are persistent for the duration of a host's session. When a host disconnects and a lower slot becomes available, new hosts do not backfill into the freed slot.

## Slot Colors

Each slot ID maps to a color, cycling through 15 values. Both the game host and the controller display this color to help players visually identify hosts.

| Slot | Color | Hex |
|------|-------|-----|
| 1 | Orange | `#FF6900` |
| 2 | Gold | `#FED000` |
| 3 | Hot Pink | `#FF2C9B` |
| 4 | Red-Pink | `#FF0066` |
| 5 | Purple | `#D500FF` |
| 6 | Olive | `#969C00` |
| 7 | Lavender | `#9B96CE` |
| 8 | Mint | `#00CD97` |
| 9 | Green | `#009B00` |
| 10 | Cyan | `#00C9FF` |
| 11 | Navy | `#112F68` |
| 12 | Lime | `#8AFF00` |
| 13 | Crimson | `#D01300` |
| 14 | Light Green | `#76D061` |
| 15 | Violet | `#7400FF` |

For slot IDs above 15, the color wraps: `colorIndex = (slotId - 1) % 15 + 1`.