# Class ID Table

Every serializable object in the protocol is identified by a 16-bit class ID. The deserializer reads this ID from the stream and uses it to dispatch to the correct object constructor.

| Class ID | Object | Description |
|----------|--------|-------------|
| 0 | BMPacket | Protocol packet wrapper |
| 1 | BMAddress | Network address (private IP + ports) |
| 2 | BMAddressList | Length-prefixed array of `BMAddress` objects |
| 3 | BMParameter | Typed value wrapper (String, Int, Float, Double, Bool, Object) |
| 4 | BMInvoke | RPC method call (method name, return method, parameters) |
| 5 | Acceleration | Accelerometer reading (x, y, z as f64) |
| 6 | TouchSet | Multi-touch input (list of Touch points) |
| 7 | IPhoneDevice | iOS device identity |
| 8 | UnityDevice | Unity host device identity |
| 9 | AckPacket | Connection request acknowledgment |
| 10 | AndroidDevice | Android device identity |
| 11 | Ping | Latency probe |
| 12 | StringLiteral | Simple string wrapper |
| 13 | Shake | Device shake event |
| 14 | BMByteChunk | Raw binary chunk with size header |
| 15 | NativeDevice | Native (C/C++) host device identity |
| 16 | PalmDevice | Palm/webOS device identity |
| 17 | ServerDevice | Registry server device identity |
| 18 | FlashDevice | Flash host device identity |
| 19 | BMRegistryInfo | Device identity + network address + slot info |
| 21 | BMArray | Ordered collection of serialized values |
| 22 | BMGyro | Gyroscope reading (x, y, z as f32) |
| 23 | Orientation | Orientation quaternion (x, y, z, w as f32) |
| 24 | DPadUpdate | D-Pad directional input (x, y as i16) |
| 256 | BMPacket | Alias for class ID 0 (BMPacket) |

## Notes

- Class ID **2** (`BMAddressList`) is assigned but is not registered by every endpoint. An endpoint that has not registered a class gives up on the whole message rather than the one field, so a class being defined somewhere does not make it safe to send everywhere.
- Class ID **13** is technically defined but never used. Shake event sending was never implemented in any of the original controller apps.
- Class ID **20** is unassigned.
- Class ID **256** (0x100) is a secondary registration for `BMPacket`, likely used as a version-specific alias.
- Device identity classes (7, 8, 10, 15, 16, 17, 18) share the same serialization format but carry different device type codes. See [Device Types](device-types.md).
