# BMParameter

`BMParameter` is a dynamically typed value wrapper used to carry arguments in [`BMInvoke`](bm-invoke.md) RPC calls. Because the protocol must support passing arbitrary types (integers, strings, nested objects, etc.) as method arguments, each value is wrapped in a `BMParameter` that prefixes it with a type tag.

- **Class ID**: 3

## Wire Format

After the [object envelope](../serialization/object-encoding.md), a `BMParameter` contains a single dynamically typed value serialized using `BMEncoding`.

The encoding consists of a **type tag** followed by the **value**:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `tag` | UTF | A single-character type tag identifying the value type. |
| 2 | `value` | (varies) | The serialized value, in the format dictated by the tag. |

### Type Tags

| Tag | Value Type | Wire Format |
|-----|-----------|-------------|
| `@` | Object | Standard [object envelope](../serialization/object-encoding.md) + class-specific payload. |
| `i` | Signed int | i32 (4 bytes) |
| `I` | Unsigned int | u32 (4 bytes) |
| `s` | Signed short | i16 (2 bytes) |
| `S` | Unsigned short | u16 (2 bytes) |
| `f` | Float | f32 (4 bytes, IEEE 754) |
| `d` | Double | f64 (8 bytes, IEEE 754) |
| `B` | Boolean | bool (1 byte) |
| `*` | String | UTF |

### Total minimum size

5 (envelope) + 3 (tag as UTF) + 1 (smallest value: bool) = **9 bytes**

## Object Parameters

When the tag is `@`, the value is a nested `Externalizable` object. This results in two consecutive `@` markers on the wire: one from the `BMEncoding` tag and one from the nested object's own envelope.

## Relationship to BMInvoke

`BMParameter` objects appear exclusively as elements of the `arguments` array inside a [`BMInvoke`](bm-invoke.md). Each argument to an RPC call is individually wrapped in its own `BMParameter`.