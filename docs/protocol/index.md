The Brass Monkey protocol allows playing games on one device (the host) and controlling them using a mobile phone (the controller).

In BM, every participant is called a Device. A Device is identified by three properties: a `deviceType` (i32), a `deviceId` (UTF string), and a `deviceName` (UTF string). See [Device Types](../reference/device-types.md) for more information on device types.

## Registry Identity

To register with the server, every device must have a `BMRegistryInfo` object. This wraps the Device identity with networking and session information:

- **Device** (nested object): Contains `deviceType`, `deviceId`, and `deviceName`.
- **BMAddress** (nested object): Contains the device's private IP address, reliable (TCP) port, and unreliable (UDP) port.
- **appId** (UTF string): A hardcoded application identifier.
- **slotId** (i16): The slot the device occupies on the registry. Only relevant for games (hosts), where the slot ID is greater than 0.
- **currentPlayers** (i16): The number of controllers currently connected to the game. Only set by games (hosts).
- **maxPlayers** (i16): The maximum number of controllers that can connect to the game. Only set by games (hosts).

## Roles

The protocol defines three roles, distinguished by `deviceType`:

- **Server** (deviceType=7): The central broker for all other devices that connect to it via TCP. It maintains a list of active games, sends the games list to the controllers, relays packets between devices. The original Brass Monkey servers are no longer operational.

- **Game Host** (deviceType=1, 3 or 5): A game running the BM SDK. It registers itself with the server, then waits for the `deviceConnectRequested` packet to start communicating with controllers. It can update the registry server with information about current players.

- **Controller** (deviceType=2, 4 or 6): A phone app. It connects to the registry server to discover available games, then requests a connection to a specific game host and communicates with it directly.

See [Device Types](../reference/device-types.md) for additional information about roles.

## Transport

Devices can communicate over two transports: **TCP** and **UDP**. Each device that accepts direct connections listens on two separate ports, both specified in its `BMAddress` object upon registration with the registry server:

- **Reliable port (TCP)**: Used for all data that must arrive in order, such as `BMInvoke` RPC calls, control scheme delivery via `BMByteChunk`, connection setup, and optionally sensor/touch data.
- **Unreliable port (UDP)**: Used for high-frequency data where low latency might matter more than guaranteed delivery, such as sensor and touch data.

The `BMReliability` enum determines which transport is used for each channel. See [Reliability Modes](../reference/reliability-modes.md) for more information.

### Framing

TCP is a byte stream with no inherent message boundaries. To delimit messages, every TCP packet is prefixed with a 4-byte little-endian length field indicating the size of the payload that follows.

UDP datagrams have inherent boundaries, so no length prefix is used. The datagram itself is the message.

!!! note "Flash Limitation"
    Flash games can only use TCP (the `Socket` class does not support UDP). Flash hosts ignore the unreliable port entirely and force all traffic to be reliable via `setReliabilityForTouch(1,1)`. As a result, all communication with a Flash host, including high-frequency touch and sensor data, happens over TCP.

### Version Handshake

The first message between any two BM devices, and a 12-byte one sent over TCP:

- **size** (i32): Always `8`, indicating that 8 bytes follow.
- **currentVersion** (i32): The sender's protocol version.
- **minVersion** (i32): The minimum protocol version the sender will accept.

Versions are encoded as `(major << 24) | (minor << 16) | build`. For example, the latest BM SDK reports `currentVersion` as `1.7.0` and `minVersion` as `0.9.0`.

Which side sends first is decided by role rather than by who dialled: a registry server opens, a game host opens toward a controller, and a controller only ever answers. Each side then checks whether its own version satisfies the other's minimum requirement, and closes the connection if not. See [Handshake](connection/handshake.md).

### Registry Connection

All devices connect to the registry server over a single TCP connection. This connection is used exclusively for `BMInvoke` RPC communication (registration, game list request, relay). The registry server never uses UDP.

## Serialization

All data is encoded in **little-endian** byte order. BM uses a custom binary serialization format.

### Object envelope

Every serializable object is wrapped in a 5-byte envelope:

| Offset | Type | Value | Description |
|--------|------|-------|-------------|
| 0 | UTF | `"@"` | Object marker (i16 length `1` + byte `0x40`) |
| 3 | i16 | varies | Class ID |

The class ID identifies the object and dictates how the remaining bytes are deserialized. See [Class ID Registry](../reference/class-ids.md) for the complete list.

### Strings

Strings are encoded as length-prefixed UTF-8:

| Offset | Type | Description |
|--------|------|-------------|
| 0 | i16 | Byte length of the UTF-8 data |
| 2 | bytes | UTF-8 encoded string |

## Packet Structure

Almost all communication is wrapped in `BMPacket` (class ID 0 or 256). After the object envelope, a `BMPacket` contains the following fields in order:

| Field | Type | Description |
|-------|------|-------------|
| channel | i32 | Channel identifier. See [Channel Types](../reference/channel-types.md). |
| sequence | i32 | Per-channel sequence number |
| timestamp | f64 | Sender timestamp in milliseconds |
| rtt | f64 | Round-trip time estimate |
| packetType | i32 | See [Packet Types](../reference/packet-types.md). |
| deviceType | i32 | Sender's device type |
| deviceId | UTF | Sender's device ID |
| deviceName | UTF | Sender's device name |
| hasMessage | bool (u8) | Whether a nested message object follows |
| message | object | Nested payload (only if hasMessage is true) |

The `message` field, when present, is itself a serialized object with its own envelope and class ID.

## Communication Model

BM uses two patterns for communication:

- **RPC via `BMInvoke`**: Structured method calls with pre-determined named methods and typed parameters. Used for registry operations, game session setup, sensor configuration, and button input. Always sent on the Message channel (3) with reliable transport.

- **Typed input objects**: Sensor and positional data (touch, acceleration, gyroscope, orientation, d-pad) are sent as dedicated object types on their own channels. Some of these can use unreliable transport depending on the configuration set by the host device. See [Channel Types](../reference/channel-types.md) and [Reliability Modes](../reference/reliability-modes.md) for more information.

## Connection Flow

At a high level, a typical session follows these steps:

1. The **game host** connects to the registry server via TCP and calls `registry.register` with its `BMRegistryInfo`.
2. A **controller** connects to the registry server via TCP, registers and calls `registry.list` to discover available games.
3. The **controller** sends a `registry.relay` containing a `deviceConnectRequested` invoke targeting the game host.
4. The **registry server** forwards the relay to the game host.
5. The **game host** opens a TCP connection to the controller, using the address the relay carried, and sends an `AckPacket` over it. It also calls `registry.update` on the server to update `currentPlayers`.
6. The **game host** delivers the `BMApplicationScheme` XML to the controller via `BMByteChunk` chunks.
7. The **controller** collects the chunks and renders the control scheme, and begins sending input data.

See [Connection Flows](../flows/lifecycle.md) for a more detailed explanation of the connection flow.