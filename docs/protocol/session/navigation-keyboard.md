# Navigation & Keyboard

Not every input is a button on the screen. When the controller is driving a menu, or standing in for a keyboard, it reports what the user does through two methods that go straight to the host. Which one it uses depends on the [control mode](set-control-mode.md) the host has put it in.

## onNavigationString

While the controller is in `NAVIGATION` mode (control mode `2`), it behaves like a d-pad for menus. Each time the user moves the focus or makes a selection, it sends one of these to the host.

| Field | Value |
|-------|-------|
| **Method** | `onNavigationString` |
| **Arguments** | 1 |

### Arguments

| # | Type | Description |
|---|------|-------------|
| 1 | `string` | The action the user took, one of the values below. |

### Navigation values

| Value | Meaning |
|-------|---------|
| `back` | Go back, or cancel. |
| `up` | Move the focus up. |
| `down` | Move the focus down. |
| `left` | Move the focus left. |
| `right` | Move the focus right. |
| `activate` | Select or confirm whatever is focused. |

## onKeyString

In `KEYBOARD` mode (control mode `1`) the controller shows a text field and lets the user type. As they type, it reports the keys to the host one at a time with this call. The host can pre-fill that field using the start string it passed to [`SetControlMode`](set-control-mode.md).

| Field | Value |
|-------|-------|
| **Method** | `onKeyString` |
| **Arguments** | 1 |

### Arguments

| # | Type | Description |
|---|------|-------------|
| 1 | `string` | The key that was pressed. |

## Behavior

Both calls are one-way: the controller is just telling the host what happened, and it doesn't expect anything back. It also only sends them in the matching mode, navigation actions in `NAVIGATION` mode and keys in `KEYBOARD` mode. In the usual `GAMEPAD` mode it sends control-scheme input instead, like [touch](../messages/touch.md), [sensors](../messages/acceleration.md), and the [d-pad](../messages/dpad-update.md).

The navigation vocabulary is fixed to the six values above, so a host can safely treat anything else as unexpected. And because keys arrive one at a time rather than as a finished string, it's up to the host to build the text back up from the run of keypresses, starting from whatever it pre-filled.
