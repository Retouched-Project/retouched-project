# Input Messages

Input messages are the objects sent by the controller to the game host during gameplay. Each message type is sent on its own dedicated channel and has a default reliability mode.

## Message Types

| Object | Class ID | Channel | Default Reliability | Description |
|--------|----------|---------|---------------------|-------------|
| [Ping](ping-ack.md) | 11 | 0 (Broadcast) | Reliable | Latency probe with device identity and address. |
| [AckPacket](ping-ack.md#ackpacket) | 9 | 0 (Broadcast) | Reliable | Connection acknowledgment with host identity and address. |
| [Acceleration](acceleration.md) | 5 | 1 (Acceleration) | Unreliable | Accelerometer reading (x, y, z). |
| [TouchSet](touch.md) | 6 | 2 (Touch) | Unreliable | Multi-touch input (list of Touch points). |
| [DPadUpdate](dpad-update.md) | 24 | 8 (DPad) | Reliable | Directional pad input (x, y). |
| [BMGyro](gyroscope.md) | 22 | 6 (Gyro) | Unreliable | Gyroscope reading (x, y, z). |
| [Orientation](orientation.md) | 23 | 7 (Orientation) | Unreliable | Orientation quaternion (x, y, z, w). |

## Notes

- Unreliable channels can be overridden to reliable via `setReliabilityForTouch`. See [Reliability Modes](../../reference/reliability-modes.md).
- Flash games always force all channels to reliable (TCP). See [Reliability Modes](../../reference/reliability-modes.md#flash-games).