# Sensor Configuration

These RPCs are sent by the game host to the controller to enable, disable, and configure sensor input channels. All are sent over the direct connection.

## Methods

| Method | Arguments | Description |
|--------|-----------|-------------|
| `enableAccelerometer` | `enabled: boolean`, `interval: float` (optional) | Enable/disable accelerometer. Interval is only sent when enabling, clamped to `[0.03, 2.0]`. |
| `enableTouch` | `enabled: boolean` | Enable or disable touch input. |
| `setTouchInterval` | `interval: float` | Set the touch sample interval in seconds. |
| `enableGyro` | `enabled: boolean` | Enable or disable gyroscope input. |
| `setGyroInterval` | `interval: float` | Set the gyroscope sample interval in seconds. |
| `enableOrientation` | `enabled: boolean` | Enable or disable orientation (rotation vector) input. |
| `setOrientationInterval` | `interval: float` | Set the orientation sample interval in seconds. |

All interval values are specified in seconds, as a `float`. Sending one as a
double instead means the call reaches no method at all and is dropped without a
reply. See [Dispatch](../objects/bm-invoke.md#dispatch).

The default interval for every sensor and for touch is `0.1`, meaning 100ms.

## enableAccelerometer

Unlike other sensors, the accelerometer combines enable and interval into a single call. When enabling, the host can optionally pass the interval as a second argument. When disabling, only the `enabled` flag is sent.

The interval is clamped on the host side:

- Minimum: `1/33` (~0.03s, 33Hz)
- Maximum: `2.0` (0.5Hz)

## Interval Conversion

The controller receives intervals as floating-point seconds and converts them to milliseconds internally (e.g., `0.1` becomes `100ms`).

## What the Interval Means

An interval is not a send rate, and it does not mean the same thing for touch as
it does for the sensors.

| Channel | When a packet goes out | Rate at the default `0.1` |
|---------|------------------------|---------------------------|
| Accelerometer, gyroscope, orientation | Once per interval | 10Hz |
| Touch | Once per **half** interval, whenever a finger has moved | 20Hz |

So touch is twice as lively as its interval suggests, and a game asking for a
particular touch rate should ask for double the period it wants. A game after
60Hz touch wants a 16ms period, so it sends `0.033`.

Touch has one more behaviour the sensors do not. A set that has already been
sent repeats at the **full** interval, up to three times, but only while touch
reliability is Unreliable. This covers a dropped datagram so a game is not left
holding a stale finger position. On a reliable channel the repeats are pointless
and do not happen.

Between flushes a controller coalesces: it keeps one position per finger and
overwrites it, so a flush carries where each finger is now and never a history
of where it has been. Raising the rate gives smoother tracking, not more data
about the past.

## Notes

- All sensor config RPCs use `BMInvoke` with a sequence ID of `1` (reliable delivery).
- The accelerometer maps to the "primary" sensor channel on the controller.
- The gyroscope maps to the "secondary" channel, and orientation to the "tertiary" channel.
- Whether gyro/orientation are available depends on the controller's [`setCapabilities`](capabilities.md) mask.