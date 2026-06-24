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
