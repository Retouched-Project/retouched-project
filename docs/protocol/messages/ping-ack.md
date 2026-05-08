# Ping / Echo / Ack / KeepAlive

These are connection health and setup messages. Unlike sensor input, they are not sent on a recurring basis during gameplay.

## Ping

A latency probe sent between peers.

- **Class ID**: 11
- **Channel**: 0 (Broadcast)
- **Packet Type**: Ping (1)

### Wire Format

After the [object envelope](../serialization/object-encoding.md), a `Ping` contains:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `deviceId` | UTF | The sender's device identifier. |
| 2 | `address` | object | A `BMAddress` (class ID `1`) with the sender's network address. |

When the receiver gets a Ping, it changes the packet type to Echo (3) and sends it back. The original `timestamp` in the `BMPacket` envelope is preserved, allowing RTT calculation.

## AckPacket

A connection acknowledgment sent by the game host to the controller after a `deviceConnectRequested` relay. Not to be confused with a TCP ACK.

- **Class ID**: 9
- **Packet Type**: Ack (2)

### Wire Format

After the [object envelope](../serialization/object-encoding.md), an `AckPacket` contains:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `device` | object | The game host's `Device` identity (class ID varies by platform). |
| 2 | `deviceAddress` | object | A `BMAddress` (class ID `1`) with the game host's connection address and ports. |

Upon receiving an Ack, the controller registers the game host and establishes a direct connection using the provided address.

## KeepAlive

A heartbeat signal sent to maintain the connection. KeepAlive packets use packet type 5 and carry no message payload (`hasMessage` is false). No response is expected.