# StringLiteral

`StringLiteral` is a simple wrapper that serializes a single UTF string as an `Externalizable` object. It is used on the String channel (4) to send plain text messages between devices.

- **Class ID**: 12

## Wire Format

After the [object envelope](../serialization/object-encoding.md), a `StringLiteral` contains:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `value` | UTF | The string content. |

### Total minimum size

5 (envelope) + 2 (empty string) = **7 bytes**