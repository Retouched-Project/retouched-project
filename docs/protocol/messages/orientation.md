# Orientation

`Orientation` carries device orientation as a quaternion from the controller.

- **Class ID**: 23
- **Channel**: 7 (Orientation)
- **Default Reliability**: Unreliable

## Wire Format

After the [object envelope](../serialization/object-encoding.md), an `Orientation` contains:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `x` | f32 | Quaternion X component. |
| 2 | `y` | f32 | Quaternion Y component. |
| 3 | `z` | f32 | Quaternion Z component. |
| 4 | `w` | f32 | Quaternion W component. |

### Total minimum size

5 (envelope) + 4 + 4 + 4 + 4 = **21 bytes**