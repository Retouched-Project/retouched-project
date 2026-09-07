# Chunked Transfer

Large payloads that exceed a single packet's practical size are split into multiple [`BMByteChunk`](../objects/bm-byte-chunk.md) objects and sent sequentially. The receiver reassembles them into the original byte array.

## Reassembly

The receiver maintains a buffer per `setId`. For each incoming chunk:

1. If `startByte` is 0 or the `setId` is new, allocate a byte array of `totalSize` bytes.
2. Copy `byteArray` into the buffer at offset `startByte`.
3. If `startByte + chunkSize == totalSize`, the transfer is complete and the assembled byte array is dispatched.

A chunk raises either `onChunkReceived` or, when step 3 says the transfer is done, `onChunkSetComplete` with the fully reassembled byte array. Never both.

Three things follow from those rules being the whole of it:

- **A receiver allocates `totalSize` before it has the bytes.** Chunk size is therefore the sender's business alone and costs the receiver nothing, while `totalSize` is what a receiver has to be willing to hold.
- **A `startByte` of 0 starts a new transfer,** discarding whatever was buffered under that `setId`. This is how a game sends `updateXML` repeatedly over one session.
- **Nothing detects a gap.** Completion is decided by one chunk ending exactly at `totalSize`, not by accounting for the bytes in between, so a chunk that never arrives leaves a run of zeros in an otherwise complete payload. Chunks are sent over the stream, which is what keeps this from happening in practice.

## Primary Use Case

The main use of chunked transfer is delivering the **control scheme XML** from the game host to the controller. The game host serializes the XML string into bytes and splits it into `BMByteChunk` objects, each sent as the `message` field of a `BMPacket` on channel 5.