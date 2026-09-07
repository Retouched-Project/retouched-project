# RequestXML & Delivery

The control scheme exchange is how the game host delivers its UI layout to the controller. The controller requests it, and the host responds with the XML data via chunked byte transfer.

## Request: RequestXML

| Field | Value |
|-------|-------|
| **Method** | `RequestXML` |
| **Return Method** | (none) |
| **Arguments** | 3 |

### Arguments

| # | Type | Description |
|---|------|-------------|
| 1 | `i32` | Controller screen height. |
| 2 | `i32` | Controller screen width. |
| 3 | `string` | Controller's `deviceId`. |

The controller sends this after connecting and reporting its capabilities via [`setCapabilities`](capabilities.md).

## Response: Chunked XML Delivery

The host does not reply with a `BMInvoke`. Instead, it delivers the XML data as a series of [`BMByteChunk`](../objects/bm-byte-chunk.md) packets on channel 5 (ByteChunk).

Each chunk contains:

- A `setId` string identifying the transfer (e.g., `"testXML"` for initial delivery, `"updateXML"` for updates).
- The byte offset, chunk size, and total size so the receiver can reassemble the full payload.
- A slice of the raw XML bytes.

The default chunk size is 10,240 bytes. If the total XML fits in a single chunk, only one is sent. Otherwise, the host sends chunks with a ~10ms delay between each.

The size is the sender's choice, not a protocol constant. Hosts raise it to 65,535 where the runtime allows, and individual games set their own. A receiver must reassemble using the offset and total size in each chunk rather than assuming any particular size.

## Completion: onControlSchemeParsed

After the controller has received and parsed all chunks, it notifies the host:

| Field | Value |
|-------|-------|
| **Method** | `onControlSchemeParsed` |
| **Arguments** | 1 |

| # | Type | Description |
|---|------|-------------|
| 1 | `string` | The controller's `deviceId`. |

Upon receiving `onControlSchemeParsed`, the host:

1. May send [`setReliabilityForTouch`](reliability-config.md) to configure the touch/sensor channel reliability.
2. Sets the device's connection state to ready.
3. Fires the `deviceLoaded` callback, signaling that the controller is fully set up and gameplay can begin.

## Post-Delivery: SetControlMode

After the XML is sent, the host sends [`SetControlMode`](set-control-mode.md) to switch the controller into the appropriate input mode for the game.

## Runtime Updates

The host can send updated control scheme XML at any time during gameplay using the same chunked delivery mechanism with a different `setId` (`"updateXML"`). This allows the game to change the controller layout dynamically (e.g., switching from a menu screen to gameplay controls).