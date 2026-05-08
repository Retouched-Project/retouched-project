# Touch / TouchSet

`TouchSet` carries multi-touch input from the controller. It contains a list of individual `Touch` points, each describing a single finger's position and state.

- **Class ID**: 6 (TouchSet)
- **Channel**: 2 (Touch)
- **Default Reliability**: Unreliable

## TouchSet Wire Format

After the [object envelope](../serialization/object-encoding.md), a `TouchSet` contains:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `count` | i32 | Number of touch points. |
| 2 | `touches` | Touch[] | Exactly `count` inline `Touch` objects (no individual envelopes). |

## Touch Fields

Each `Touch` is serialized inline (without its own object envelope) in the following order:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `x` | f32 | Horizontal position on the touch surface. |
| 2 | `y` | f32 | Vertical position on the touch surface. |
| 3 | `screenWidth` | i16 | Width of the controller's touch area. |
| 4 | `screenHeight` | i16 | Height of the controller's touch area. |
| 5 | `state` | i32 | Touch state (see below). |
| 6 | `id` | i32 | Unique identifier for this touch point (finger). |

## Touch States

| Value | State | Description |
|-------|-------|-------------|
| 1 | Began | A new touch has started. |
| 2 | Moved | The touch point has moved since the last update. |
| 3 | Stationary | The touch point has not moved. |
| 4 | Ended | The touch has been released. |
| 5 | Cancelled | The touch was cancelled (e.g., interrupted by a system event). |

## Note on inline serialization

`Touch` objects are written directly by calling `writeExternal()` on each one, without wrapping them in an object envelope. This means `Touch` does not appear as a standalone serialized object on the wire. It has no usable class ID (returns `-1`).