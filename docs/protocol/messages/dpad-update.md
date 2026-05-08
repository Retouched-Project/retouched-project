# D-Pad Update

`DPadUpdate` represents directional pad input from the controller.

- **Class ID**: 24
- **Channel**: 8 (DPad)
- **Default Reliability**: Reliable

## Wire Format

After the [object envelope](../serialization/object-encoding.md), a `DPadUpdate` contains:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `x` | i16 | Horizontal direction. |
| 2 | `y` | i16 | Vertical direction. |

### Total minimum size

5 (envelope) + 2 + 2 = **9 bytes**