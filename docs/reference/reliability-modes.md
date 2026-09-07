# Reliability Modes

The reliability mode determines which transport a packet is sent over. It acts as a transport selector, not an abstract delivery guarantee.

| Code | Name | Transport | Description |
|------|------|-----------|-------------|
| 0 | Unreliable | UDP | Packet is sent over UDP. May be lost, duplicated, or reordered. |
| 1 | ReliableUnordered | TCP | Packet is sent over TCP. Delivery is guaranteed. |
| 2 | ReliableOrdered | TCP | Packet is sent over TCP. Delivery is guaranteed. |

!!! note
    `ReliableUnordered` (1) and `ReliableOrdered` (2) select the same transport, and TCP orders the bytes either way, so the two are interchangeable when choosing how to send something.

## Reliability Is Never Sent

The mode does not appear anywhere in the [`BMPacket`](../protocol/packets/bm-packet.md) wire format. It is a local value at both ends, and it means something different at each.

**On the sending side** it selects the socket, which is all it does.

**On the receiving side** it is set by whichever reader accepted the bytes, so it records how the packet arrived rather than anything the sender declared. A stream reader stamps every packet it decodes as `ReliableOrdered`; a datagram reader leaves it at the default of `Unreliable`.

Two behaviours read it, and both are asking a local question:

- A controller adopts the source port of an incoming Ack as the sender's reliable port only when that Ack came in over the stream, which is the reader's stamp rather than the sender's choice.
- A channel discards an arriving packet whose `sequence` is lower than the last one seen, unless that channel is configured as `ReliableOrdered`. This reads the mode the channel was last given by [`setReliabilityForTouch`](../protocol/session/reliability-config.md), not anything on the packet.

## Transport Mapping

- **Unreliable (0)** = send over UDP.
- **Reliable (1 or 2)** = send over TCP.

There is no "reliable UDP" or "unreliable TCP". The reliability mode is a direct selector for the underlying transport. Because of this split, each device's `BMAddress` object announces two distinct ports: one dedicated to reliable TCP traffic, and another for unreliable UDP traffic.

## Flash Games

Flash Player's `Socket` class only supports TCP. Flash games have no access to UDP transport (unless using Adobe AIR native extensions, which the Brass Monkey SDK did not use). As a result, Flash games always call `setReliabilityForTouch(1,1)`, forcing all traffic (including touch and sensor data) through TCP. The Flash SDK also ignores the unreliable port entirely (never stores it on receive).

## Typical Usage

| Channel | Default Reliability | Transport | Notes |
|---------|---------------------|-----------|-------|
| Message (RPC) | Reliable (1 or 2) | TCP | Always reliable |
| Touch | Unreliable (0) | UDP | Overridden to Reliable for Flash games |
| Acceleration | Unreliable (0) | UDP | Overridden to Reliable for Flash games |
| Gyro | Unreliable (0) | UDP | Overridden to Reliable for Flash games |
| Orientation | Unreliable (0) | UDP | Overridden to Reliable for Flash games |
| D-Pad | Reliable (1 or 2) | TCP | Always reliable |
| Bytes | Reliable (1 or 2) | TCP | Always reliable |
