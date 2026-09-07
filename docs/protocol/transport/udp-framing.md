# UDP Framing

UDP datagrams have inherent message boundaries, so no length prefix is required. Each datagram contains exactly one serialized object.

## Wire Layout

```
[envelope (5 bytes)] [object fields...]
```

The datagram payload begins directly with the [object envelope](../serialization/object-encoding.md). There is no length prefix.

## Reading

The receiver allocates a 1024-byte buffer per read. Upon receiving a datagram, the entire buffer is passed to the deserializer, which reads the object envelope and class-specific fields.

That buffer is the ceiling for anything sent unreliably. A datagram longer than it is truncated on arrival rather than refused, so the deserializer meets a short object and fails to parse it. The input objects that travel this way are tens of bytes each and never come close, but nothing larger can use this path.

## Usage

UDP is used for high-frequency, latency-sensitive data such as touch, acceleration, gyroscope, and orientation input. These channels default to unreliable transport. See [Reliability Modes](../../reference/reliability-modes.md) and [Channel Types](../../reference/channel-types.md).

UDP traffic always targets the `unreliablePort` specified in the remote device's `BMAddress`. See [BMRegistryInfo](../objects/bm-registry-info.md).

## Flash Limitation

Flash Player's `Socket` class does not support UDP. Flash games force all traffic through TCP by overriding the reliability mode. As a result, UDP is never used when communicating with a Flash host. See [Reliability Modes](../../reference/reliability-modes.md#flash-games).