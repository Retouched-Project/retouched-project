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
| 1 | `device` | object | The connecting controller's `Device` identity (class ID varies by platform). |
| 2 | `deviceAddress` | object | A `BMAddress` (class ID `1`) with the **game host's own** address and ports. |

The two fields point in opposite directions, which is easy to get backwards. The
identity is the recipient's and the address is the sender's.

Upon receiving an Ack, the controller registers the game host and direct
communication continues.

### What the controller keeps

A controller does not take the Ack at face value. It builds the peer identity
from the `BMPacket` header rather than from field 1, so `device` is written by
every implementation and read by none.

Of `deviceAddress`, only one field survives:

| Field | What the controller does with it |
|-------|----------------------------------|
| `address` | Replaced with the host the packet actually arrived from. |
| `reliablePort` | Replaced with the observed source port, but only when the Ack arrived over the stream rather than as a datagram. |
| `unreliablePort` | Kept exactly as sent. Never verified. |

So the unreliable port is the only part of the address a game gets to declare,
and the only part it can get wrong without the transport correcting it.

### Why the Ack is required

A controller treats a game as unreachable until that game has acked it, and
ignores data packets from any device it has not been acked by. Nothing else
makes a game reachable, so a controller cannot send anything at all, not even a
Ping, before the Ack arrives.

This is the reverse of what the connection direction suggests. The game dials
the controller, and then still has to speak first. A game that connects and
waits will wait forever.

## KeepAlive

A heartbeat signal sent to maintain the connection. KeepAlive packets use packet type 5 and carry no message payload (`hasMessage` is false). No response is expected.