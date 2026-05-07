# BMPacket

`BMPacket` is the fundamental wrapper for nearly all communication in BM. Every message exchanged between devices is serialized inside a `BMPacket`.

- **Class ID**: 0 (alias: 256)
- **Flash SDK name**: `Packet`

## Wire Format

After the [object envelope](../serialization/object-encoding.md), a `BMPacket` contains the following fields in order:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `channel` | i32 | Determines the type of payload. See [Channel Types](../../reference/channel-types.md). |
| 2 | `sequence` | i32 | Per-channel sequence counter. See [Channels](channels.md). |
| 3 | `timestamp` | f64 | Sender's time in milliseconds (epoch). |
| 4 | `rtt` | f64 | Round-trip time estimate in milliseconds. |
| 5 | `packetType` | i32 | See [Packet Types](../../reference/packet-types.md). |
| 6 | `deviceType` | i32 | Identifies the sender's role. See [Device Types](../../reference/device-types.md). |
| 7 | `deviceId` | UTF | Sender's persistent device identifier. |
| 8 | `deviceName` | UTF | Sender's human-readable device name (e.g. model name). |
| 9 | `hasMessage` | bool | Whether a nested message object follows. |
| 10 | `message` | object | The nested payload. Only present if `hasMessage` is true. |

### Total minimum size

Without a nested message: 5 (envelope) + 4 + 4 + 8 + 8 + 4 + 4 + 2 + 2 + 1 = **42 bytes**
(assuming empty `deviceId` and `deviceName`).

## Sequence Numbers

Each channel maintains its own independent sequence counter. When a packet is sent, the sequence number is drawn from that channel's counter, not a shared global one. See [Channels](channels.md) for more details.

If a packet is constructed without a channel assignment (i.e., `sequence` is still `0` at serialization time), a global fallback counter is used instead. This can occur for packets sent outside the normal channel pipeline, such as `Ack` or `Ping` responses.

## Packet Types

See [Packet Types](../../reference/packet-types.md) for a detailed breakdown.

## Message Field

When `hasMessage` is true, the `message` field is a serialized object with its own [object envelope](../serialization/object-encoding.md) and class ID. The type of object depends on the channel. See [Channel Types](../../reference/channel-types.md) for a detailed breakdown.

## Class ID Alias

`BMPacket` is registered under both class ID **0** and **256** in the Android SDK. Both produce the same object. The alias presumably exists for forward compatibility.