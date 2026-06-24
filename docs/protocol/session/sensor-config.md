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

All interval values are specified in seconds. The default interval for all sensors is `0.1` (100ms / 10Hz).

## enableAccelerometer

Unlike other sensors, the accelerometer combines enable and interval into a single call. When enabling, the host can optionally pass the interval as a second argument. When disabling, only the `enabled` flag is sent.

The interval is clamped on the host side:

- Minimum: `1/33` (~0.03s, 33Hz)
- Maximum: `2.0` (0.5Hz)

## Interval Conversion

The controller receives intervals as floating-point seconds and converts them to milliseconds internally (e.g., `0.1` becomes `100ms`).

## Notes

- All sensor config RPCs use `BMInvoke` with a sequence ID of `1` (reliable delivery).
- The accelerometer maps to the "primary" sensor channel on the controller.
- The gyroscope maps to the "secondary" channel, and orientation to the "tertiary" channel.
- Whether gyro/orientation are available depends on the controller's [`setCapabilities`](capabilities.md) mask.