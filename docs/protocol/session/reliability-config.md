# setReliabilityForTouch

Configures the transport reliability mode for touch and sensor input channels. Sent by the game host to the controller over the direct connection.

## Request

| Field | Value |
|-------|-------|
| **Method** | `setReliabilityForTouch` |
| **Arguments** | 2 |

### Arguments

| # | Type | Description |
|---|------|-------------|
| 1 | `i32` | Touch reliability mode. Applied to the touch input channel. |
| 2 | `i32` | Control reliability mode. Applied to all sensor channels (accelerometer, gyroscope, orientation). |

## Reliability Values

| Value | Name | Transport |
|-------|------|-----------|
| `0` | Unreliable | UDP |
| `1` | ReliableUnordered | TCP |
| `2` | ReliableOrdered | TCP |

See [Reliability Modes](../../reference/reliability-modes.md) for full details.

## Behavior

When the controller receives this RPC, it updates the transport used for outgoing input packets:

- **Touch reliability** (argument 1) sets the transport for touch/`TouchSet` packets. When set to `0`, touch data is sent over UDP for minimal latency. When set to `1` or `2`, touch data is sent over TCP for guaranteed delivery.
- **Control reliability** (argument 2) sets the transport for all sensor channels: accelerometer, gyroscope, and orientation. This single value is applied uniformly to the primary (accelerometer), secondary (gyroscope), and tertiary (orientation) channels.

The default reliability for both touch and sensor channels is `0` (Unreliable / UDP). This RPC overrides those defaults for the duration of the session.

## Channel Mapping

| Channel | Controlled By |
|---------|---------------|
| Touch | Argument 1 (`touchReliability`) |
| Accelerometer | Argument 2 (`controlReliability`) |
| Gyroscope | Argument 2 (`controlReliability`) |
| Orientation | Argument 2 (`controlReliability`) |

!!! note
    D-Pad input and RPC messages are always sent reliably over TCP, regardless of this setting. This RPC only affects high-frequency input channels where UDP is the default.

## Reliability Is a Sender's Concern

This setting changes how a controller **sends**. It changes nothing about how a
game **receives**.

A receiving endpoint routes an incoming packet by its channel number alone. It
never asks which socket delivered it, and it decodes a `TouchSet` that arrived
over TCP exactly as it would one that arrived over UDP. The only receive-side
effect reliability has at all is on an unordered channel, where a packet whose
sequence number is older than the last one seen is discarded, and that is a
comparison of numbers rather than of transports.

Three things follow, and they account for most cases where input appears to go
missing:

- A controller that has no UDP socket sends on TCP whatever it was asked for,
  and the game receives everything normally. Nothing has to be negotiated.
- A game that never calls this RPC still receives touch and sensor data that
  arrives reliably. There is no need to "accept" TCP input.
- A game that asks for unreliable input and then does not open a UDP port
  receives nothing on those channels. The datagrams are addressed correctly, so
  nothing rejects them; they simply arrive at a port where no socket is
  listening, and are dropped by the operating system before any part of the
  protocol sees them.

The last one is the only genuine failure mode here, and it is not a disagreement
between the two sides. It is a game advertising a port it never bound. See
[Registry Register](../registry/register.md) for where that port is declared.

## Flash Games

Flash Player's `Socket` class only supports TCP. Flash-based games always call `setReliabilityForTouch(1, 1)` to force all input through the TCP channel. Since Flash games never open a UDP port, the unreliable transport is unavailable, and this call ensures the controller does not attempt to send packets over a non-existent UDP connection.

## Typical Calls

| Game Type | Call | Effect |
|-----------|------|--------|
| Any (default) | `setReliabilityForTouch(0, 0)` | Touch and sensors over UDP. Lowest latency. |
| Flash | `setReliabilityForTouch(1, 1)` | All input over TCP. Required for Flash compatibility. |
| Any (reliable touch) | `setReliabilityForTouch(1, 0)` | Touch over TCP, sensors over UDP. Used when the host requires guaranteed touch delivery. |
| Any (reliable sensors) | `setReliabilityForTouch(0, 1)` | Touch over UDP, sensors over TCP. Used when the host requires guaranteed sensor delivery. |

## Notes

- This RPC does not have a response. The controller applies the settings immediately.
- If a game does not send this RPC, the controller uses the default reliability of `0` (UDP) for both touch and sensor channels.
- It can be sent at any point in a session, as many times as a game likes. There
  is no connection state that has to be reached first and nothing is restarted;
  the new setting simply applies from the next packet onward. A game is free to
  move touch onto TCP partway through and back again.
- Both arguments are plain signed integers, tag `i`. Unlike the sensor
  intervals, there is no float involved here.

## Asymmetry

Only a game can send this RPC. There is no equivalent in the other direction, so
a controller has no way to tell a game how it would prefer to be treated.

That gives the two sides very different room for error when declaring an
unreliable port during registration. A game that declares a port it does not
have can correct the consequence itself, by calling this RPC and moving its
input onto the stream. A controller that declares a port it does not have has no
such recourse, and no way to say so afterwards. Its declaration has to be
truthful the first time.
