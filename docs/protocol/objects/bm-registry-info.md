# BMRegistryInfo

`BMRegistryInfo` is the identity payload sent by every device during registration with the registry server. It bundles the device's identity, network address, application identifier, and session metadata into a single object.

- **Class ID**: 19

## Wire Format

After the [object envelope](../serialization/object-encoding.md), a `BMRegistryInfo` contains the following fields in order:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `device` | object | A `Device` object (class ID varies by platform). Contains the device's identity. |
| 2 | `deviceAddress` | object | A `BMAddress` object (class ID `1`). Contains the device's network address and ports. |
| 3 | `appId` | UTF | Application identifier string. |
| 4 | `slotId` | i16 | The slot this device occupies on the registry. Controllers use `0`; hosts use a positive value assigned by the server. |
| 5 | `currentPlayers` | i16 | Number of controllers currently connected. **Only present if `slotId > 0` (host).** |
| 6 | `maxPlayers` | i16 | Maximum number of controllers allowed. **Only present if `slotId > 0` (host).** |

### Total minimum size

5 (envelope) + 5 (Device envelope) + 5 (BMAddress envelope) = **15 bytes** minimum for the envelopes alone, plus the field data of each nested object.

## Nested Objects

### Device

The `device` field is a platform-specific `Device` subclass. All subclasses share the same wire format:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `deviceType` | i32 | See [Device Types](../../reference/device-types.md). |
| 2 | `deviceId` | UTF | Persistent device identifier. |
| 3 | `deviceName` | UTF | Human-readable device name (e.g., model name). |

The class ID in the envelope identifies the platform. See [Class ID Table](../../reference/class-ids.md) for the full list of device class IDs.

### BMAddress

The `deviceAddress` field contains the device's private network address:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `address` | UTF | Private IP address (e.g., `"192.168.1.124"`). |
| 2 | `unreliablePort` | i32 | UDP port for unreliable traffic. |
| 3 | `reliablePort` | i32 | TCP port for reliable traffic. |

- **Class ID**: 1

## Conditional Fields

The `currentPlayers` and `maxPlayers` fields are only serialized when `slotId > 0`, which indicates the device is a game host. Controllers (which always have `slotId = 0`) omit these fields entirely. The deserializer uses the same `slotId > 0` check to decide whether to read them.