# Control Modes

Control modes determine what the controller app displays to the user. The game sets the control mode via the `SetControlMode` RPC.

| Ordinal | Name | Description |
|---------|------|-------------|
| 0 | GAME | The main gameplay mode. The controller renders the control scheme UI (buttons, D-Pad, touch surface) and sends input to the game. |
| 1 | TEXT | Text input mode. The controller shows a keyboard and sends typed text to the game. An optional `startString` pre-populates the input field. |
| 2 | NAV | Navigation mode. The controller shows a simplified navigation interface (back, home, select). |
| 3 | WAIT | Waiting mode. The controller shows a "waiting for game" screen, used while another host is expected to take over. |

## Notes

- **A session begins in `GAME` without being told to.** The mode is set when the session is created, so a host that never sends `SetControlMode` still gets a working gamepad. Not every host sends one, and none has to.
- The game can switch modes at any time by sending `SetControlMode` with the ordinal and an optional string parameter, which is only meaningful for **TEXT** mode.
- [`WaitForNewHost`](../protocol/session/trial-purchase.md) moves the controller into **WAIT** as part of handling it, without a separate `SetControlMode`.
- An ordinary disconnection does not change the mode. The controller leaves the session altogether and returns to choosing a host. **WAIT** is for a handover that is expected to complete, not for the absence of a game.
