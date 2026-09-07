# TCP Framing

TCP is a byte stream with no inherent message boundaries. To delimit individual messages, every TCP message is prefixed with a 4-byte little-endian length field.

## Wire Layout

```
[length (i32)] [payload...]
|<- 4 bytes ->|<- length bytes ->|
```

The length field specifies the number of bytes that follow, excluding the length field itself.

## Payload Types

Two types of messages are sent over TCP:

1. **Handshake**: The first message on any new TCP connection. The length is always `8` (two i32 fields). See [Handshake](../connection/handshake.md).
2. **BMPacket**: All subsequent messages. The payload begins with the standard [object envelope](../serialization/object-encoding.md) and contains a serialized `BMPacket`.

The receiver distinguishes between these based on connection state: the first message is always interpreted as a handshake, and all subsequent messages are interpreted as `BMPacket` objects.

## Reading

The receiver buffers incoming bytes until at least 4 bytes are available. It then reads the length prefix, continues buffering until `length` bytes have been received, and dispatches the complete message for deserialization.

If the incoming data contains more bytes than one message, the receiver processes messages sequentially in a loop until the buffer is exhausted.

## Buffer Management

The TCP socket uses a dynamically resized buffer. If the announced length exceeds the current buffer capacity, the buffer is expanded, but only while the required capacity stays under 98,304 bytes. A message needing more than that is **not** dropped and raises no error.

!!! warning
    A receiver asked to grow past that limit simply does not grow. It goes on waiting for a message that can never fit, so the connection stops making progress and nothing is logged at either end. The sender's write succeeds and the receiver stays silent.

    The length prefix counts toward the limit, so the largest payload that reliably fits is a few bytes under 98,300. A sender that splits large content into [chunks](../packets/chunked-transfer.md) never approaches it, which is what the chunk mechanism is for.

## Flash Socket Policy

Before any BM traffic, a Flash client may send a socket policy request on the same TCP port. The receiver detects this by checking if the incoming bytes match the `<policy-file-request/>` prefix. If so, it responds with a cross-domain policy XML and closes the connection. See [Flash Socket Policy](flash-policy.md).