# BMArray

`BMArray` is an ordered collection of dynamically typed values. Like [`BMParameter`](bm-parameter.md), each element is serialized using `BMEncoding`, allowing the array to hold a mix of primitives, strings, and nested objects.

- **Class ID**: 21

## Wire Format

After the [object envelope](../serialization/object-encoding.md), a `BMArray` contains:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `length` | i16 | The number of elements in the array. |
| 2 | `elements` | (varies) | Exactly `length` values, each serialized using `BMEncoding`. |

Each element follows the same encoding as a [`BMParameter`](bm-parameter.md) value: a UTF type tag followed by the value in the format dictated by that tag. See [BMParameter: Type Tags](bm-parameter.md#type-tags) for the full tag table.

### Total minimum size

5 (envelope) + 2 (length) = **7 bytes** (empty array).

## Differences from BMParameter

| | BMParameter | BMArray |
|---|---|---|
| Class ID | 3 | 21 |
| Holds | A single value | Multiple values |
| Length field | None (always 1 value) | i16 prefix |
| Element encoding | BMEncoding | BMEncoding (per element) |

## Note on length type

`BMArray` uses an **i16** for its length field, unlike `BMInvoke`'s argument count which uses an **i32**. This limits a `BMArray` to 32,767 elements.