# Acceleration

`Acceleration` carries accelerometer sensor data from the controller.

- **Class ID**: 5
- **Channel**: 1 (Acceleration)
- **Default Reliability**: Unreliable

## Wire Format

After the [object envelope](../serialization/object-encoding.md), an `Acceleration` contains:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `x` | f64 | Acceleration along the X axis. |
| 2 | `y` | f64 | Acceleration along the Y axis. |
| 3 | `z` | f64 | Acceleration along the Z axis. |

### Total minimum size

5 (envelope) + 8 + 8 + 8 = **29 bytes**

## Note

Acceleration uses f64 (double precision), unlike gyroscope and orientation which use f32.