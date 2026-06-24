# vibrate

A simple nudge: this tells the controller to buzz. The host sends it to the controller over the direct connection, and there's nothing to dial in: no duration, no strength, no pattern.

## Request

| Field | Value |
|-------|-------|
| **Method** | `vibrate` |
| **Arguments** | 0 |

## Behavior

The call carries no arguments and gets no reply. On the original controller every `vibrate` produces exactly the same buzz: a fixed one-second vibration. There's no field to make it shorter or softer, and the controller gives the user no way to turn it off either, so whenever the host sends `vibrate`, the phone buzzes for a full second. A game that wants a longer or repeated rumble simply sends the call again.
