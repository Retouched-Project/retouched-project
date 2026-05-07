# Handshake

The version handshake is the first message exchanged between any two BM devices over TCP. It occurs before any `BMPacket` traffic and is not wrapped in the standard object envelope.

## Wire Format

The handshake is a fixed 12-byte message:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `size` | i32 | Always `8`. The number of bytes that follow. |
| 2 | `currentVersion` | i32 | The sender's protocol version. |
| 3 | `minVersion` | i32 | The minimum protocol version the sender will accept from the remote peer. |

Both sides send this message independently upon TCP connection. There is no request/response ordering; each peer writes its handshake and then reads the other's.

## Version Encoding

Versions are packed into a single i32 using bit shifting:

`(major << 24) | (minor << 16) | build`

For example, version `1.7.0` encodes as `0x01070000` and version `0.9.0` encodes as `0x00090000`.

The latest SDK reports:

- **currentVersion**: `1.7.0`
- **minVersion**: `0.9.0`

## After the Handshake

Once both sides have exchanged version messages, each peer checks compatibility. See [Version Compatibility](version-compat.md) for the rules. If the versions are compatible, the connection proceeds to normal `BMPacket` traffic. If not, the connection is closed immediately.