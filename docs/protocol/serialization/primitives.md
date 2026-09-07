# Primitive Types

All primitive values are encoded in **little-endian** byte order. This page documents every primitive type used in BM serialization.

## Integer Types

| Type | Size | Range | Notes |
|------|------|-------|-------|
| i16 | 2 bytes | -32,768 to 32,767 | Signed. Used for class IDs, string lengths, `slotId`, player counts. |
| u16 | 2 bytes | 0 to 65,535 | Unsigned. |
| i32 | 4 bytes | -(2<sup>31</sup>) to (2<sup>31</sup>)-1 | Signed. Used for `channel`, `sequence`, `packetType`, `deviceType`, `ports`. |
| u32 | 4 bytes | 0 to (2<sup>32</sup>)-1 | Unsigned. |

## Floating Point Types

| Type | Size | Format | Notes |
|------|------|--------|-------|
| f32 | 4 bytes | IEEE 754 single | Used in sensor data (touch, gyroscope, orientation). |
| f64 | 8 bytes | IEEE 754 double | Used for `timestamp` and `rtt` in `BMPacket`, and acceleration sensor data. |

## Boolean

| Type | Size | Encoding |
|------|------|----------|
| bool | 1 byte | `0x00` = false, any non-zero value = true |

Used for `hasMessage` in `BMPacket` and value encoding in `BMParameter`.

## Byte

| Type | Size | Notes |
|------|------|-------|
| u8 | 1 byte | Raw byte. Used in `BMByteChunk` payloads and as part of the object envelope. |

## UTF String

Strings are length-prefixed UTF-8:

| Offset | Type | Description |
|--------|------|-------------|
| 0 | i16 | Byte length of the UTF-8 data that follows |
| 2 | bytes | UTF-8 encoded string data |

A null or empty string is encoded as `00 00` (length 0, no data bytes).

!!! note
    The length prefix is the **byte length** of the encoded UTF-8 data, not the character count. Multi-byte UTF-8 characters will have a byte length greater than the character count.

!!! warning
    The length prefix is little-endian, like every other integer here. The
    conventional UTF writing call on several platforms emits a **big-endian**
    length instead, so reaching for it produces strings no endpoint can read.

Characters outside the Basic Multilingual Plane are written as two three byte
sequences rather than one four byte sequence, and readers decode the payload as
plain UTF-8 without reversing that, so they do not survive a round trip.
Everything within the BMP is ordinary UTF-8.