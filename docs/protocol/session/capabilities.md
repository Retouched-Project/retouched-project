# setCapabilities

Reports the controller's hardware sensor capabilities to the game host. Sent by the controller immediately after establishing a direct connection.

## Request

| Field | Value |
|-------|-------|
| **Method** | `setCapabilities` |
| **Arguments** | 1 |

### Arguments

| # | Type | Description |
|---|------|-------------|
| 1 | `i32` | A bitmask of supported hardware sensors. |

## Capability Bits

| Bit | Value | Capability |
|-----|-------|------------|
| 0 | `1` | Gyroscope supported. |
| 1 | `2` | Orientation (rotation vector) supported. |

The game host reads these two flags to decide which sensor RPCs to send during gameplay, for example only enabling the gyroscope when bit 0 is set.

## Behavior

The controller computes the mask by checking which sensor channels are available on the device:

- If the gyroscope sensor is present, bit 0 is set.
- If the orientation (rotation vector) sensor is present, bit 1 is set.

A controller with both sensors reports a mask of `3`. A controller with neither reports `0`.
The host does not respond to this call. It stores the mask on the device object and may use it to conditionally enable gyroscope or orientation input.