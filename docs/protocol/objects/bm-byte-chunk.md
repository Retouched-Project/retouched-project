# BMByteChunk

`BMByteChunk` carries a segment of a larger binary payload. Multiple chunks with the same `setId` are reassembled by the receiver into the complete byte array. See [Chunked Transfer](../packets/chunked-transfer.md) for the reassembly protocol.

- **Class ID**: 14
- **Channel**: 5 (Bytes)
- **Reliability**: Always reliable (TCP)

## Wire Format

After the [object envelope](../serialization/object-encoding.md), a `BMByteChunk` contains:

| # | Field | Type | Description |
|---|-------|------|-------------|
| 1 | `setId` | UTF | Identifier grouping all chunks belonging to the same transfer. |
| 2 | `startByte` | i32 | Byte offset of this chunk within the complete payload. |
| 3 | `chunkSize` | i32 | Number of payload bytes in this chunk. |
| 4 | `totalSize` | i32 | Total size of the complete payload in bytes. |
| 5 | `byteArray` | bytes | The raw chunk data (`chunkSize` bytes). |

### Total minimum size

5 (envelope) + 2 (empty setId) + 4 + 4 + 4 = **19 bytes** (with zero-length payload).