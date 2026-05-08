# Gyroscope

`BMGyro` carries gyroscope sensor data from the controller.

- **Class ID**: 22
- **Channel**: 6 (Gyro)
- **Default Reliability**: Unreliable

## Wire Format

After the [object envelope](../serialization/object-encoding.md), a `BMGyro` contains:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `x` | f32 | Rotation rate around the X axis. |
| 2 | `y` | f32 | Rotation rate around the Y axis. |
| 3 | `z` | f32 | Rotation rate around the Z axis. |

### Total minimum size

5 (envelope) + 4 + 4 + 4 = **17 bytes**