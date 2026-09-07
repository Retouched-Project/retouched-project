# SetControlMode

Instructs the controller to switch its input mode. Sent by the game host to the controller over the direct connection.

## Request

| Field | Value |
|-------|-------|
| **Method** | `SetControlMode` |
| **Arguments** | 1 or 2 |

### Arguments

| # | Type | Description |
|---|------|-------------|
| 1 | `i32` | The control mode ordinal. See table below. |
| 2 | `string` | *(Optional)* Initial text content. Only used with `TEXT` mode. |

## Control Modes

| Value | Name | Description |
|-------|------|-------------|
| `0` | GAME | Standard gameplay mode. The controller renders the XML control scheme and sends touch/sensor input. |
| `1` | TEXT | Text input mode. The controller displays a text field, optionally pre-filled with the `startString` argument. |
| `2` | NAV | System navigation mode. The controller sends navigation events (e.g., back, home) via `onNavigationString`. |
| `3` | WAIT | Idle/loading mode. The controller shows a waiting state and does not send input. |

## Usage

The host typically sends `SetControlMode(GAME)` after the control scheme XML has been delivered and the controller has confirmed parsing via `onControlSchemeParsed`.

During gameplay, the host can switch modes dynamically, for example, switching to `TEXT` mode when a text input field is focused, or to `WAIT` mode during a loading screen.