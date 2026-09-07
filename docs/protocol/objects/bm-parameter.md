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

| Tag | Value Type | Wire Format | Safe to send |
|-----|-----------|-------------|------------------------|
| `@` | Object | Standard [object envelope](../serialization/object-encoding.md) + class-specific payload. | Yes |
| `i` | Signed int | i32 (4 bytes) | Yes |
| `I` | Unsigned int | u32 (4 bytes) | Yes, but see below |
| `s` | Signed short | i16 (2 bytes) | No |
| `S` | Unsigned short | u16 (2 bytes) | No |
| `f` | Float | f32 (4 bytes, IEEE 754) | Yes |
| `d` | Double | f64 (8 bytes, IEEE 754) | No |
| `B` | Boolean | bool (1 byte) | Yes |
| `*` | String | UTF | Yes |

### Total minimum size

5 (envelope) + 3 (tag as UTF) + 1 (smallest value: bool) = **9 bytes**

## Choosing a Tag

Every endpoint can decode all nine tags, so the table above is the full receiving
surface. The sending surface is smaller, and staying inside it matters, because
the tag decides which method the receiver calls. See
[Dispatch](bm-invoke.md#dispatch).

In practice only `i`, `I`, `f`, `B`, `*` and `@` reach the wire. `S` never
appears at all.

`s` and `d` are well formed, and an implementation is free to emit either, but
no method anywhere declares a short or a double parameter, so a value sent under
one matches nothing and the call is dropped. This is the usual surprise with
fractional values: a double is not refused on the way out, it is
undeliverable when it lands. Every fractional argument in the protocol is a
float, and `f` is the only tag that reaches one.

`I` deserves its own warning. It is the correct tag for an unsigned value, but
on some endpoints it decodes to a **boxed** integer type rather than a
primitive, and a boxed type matches no primitive signature. An `I` argument can
therefore fail to reach a method that looks like it should accept it. `i` is the
safer choice: it is the same four bytes on the wire, differing only in the tag,
and it widens cleanly to whatever the receiving method declares.

!!! warning
    Sending a value that is technically valid but wider than the target method
    expects is not an error anyone will see. The call is dropped on the far
    side, silently, and the sender's write appears to succeed.

## Object Parameters

When the tag is `@`, the value is a nested `Externalizable` object. This results in two consecutive `@` markers on the wire: one from the `BMEncoding` tag and one from the nested object's own envelope.

## Relationship to BMInvoke

`BMParameter` objects appear exclusively as elements of the `arguments` array inside a [`BMInvoke`](bm-invoke.md). Each argument to an RPC call is individually wrapped in its own `BMParameter`.