# Chunked Transfer

Large payloads that exceed a single packet's practical size are split into multiple [`BMByteChunk`](../objects/bm-byte-chunk.md) objects and sent sequentially. The receiver reassembles them into the original byte array.

## Reassembly

The receiver maintains a buffer per `setId`. For each incoming chunk:

1. If `startByte` is 0 or the `setId` is new, allocate a byte array of `totalSize` bytes.
2. Copy `byteArray` into the buffer at offset `startByte`.
3. If `startByte + chunkSize == totalSize`, the transfer is complete and the assembled byte array is dispatched.

Individual chunks trigger an `onChunkReceived` callback. The final chunk triggers `onChunkSetComplete` with the fully reassembled byte array.

## Primary Use Case

The main use of chunked transfer is delivering the **control scheme XML** from the game host to the controller. The game host serializes the XML string into bytes and splits it into `BMByteChunk` objects, each sent as the `message` field of a `BMPacket` on channel 5.