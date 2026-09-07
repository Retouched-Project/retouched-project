# Navigation & Keyboard

Not every input is a button on the screen. When the controller is driving a menu, or standing in for a keyboard, it reports what the user does through two methods that go straight to the host. Which one it uses depends on the [control mode](set-control-mode.md) the host has put it in.

## onNavigationString

While the controller is in `NAV` mode (control mode `2`), it behaves like a d-pad for menus. Each time the user moves the focus or makes a selection, it sends one of these to the host.

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

In `TEXT` mode (control mode `1`) the controller shows a text field and lets the user type. The host can pre-fill that field using the start string it passed to [`SetControlMode`](set-control-mode.md), and the controller reports each edit as it happens with this call.

| Field | Value |
|-------|-------|
| **Method** | `onKeyString` |
| **Arguments** | 1 |

### Arguments

| # | Type | Description |
|---|------|-------------|
| 1 | `string` | One incremental edit to the text. See the values below. |

### Edit Values

| Value | Meaning |
|-------|---------|
| a non-empty string | Text that was just inserted (one character while typing, or several when text is pasted). |
| an empty string (`""`) | One character was deleted (a backspace). |
| `"\n"` | The Enter / submit key. |

## Behavior

Both calls are one-way: the controller is just telling the host what happened, and it doesn't expect anything back. It also only sends them in the matching mode, navigation actions in `NAV` mode and keys in `TEXT` mode. In the usual `GAME` mode it sends control-scheme input instead, like [touch](../messages/touch.md), [sensors](../messages/acceleration.md), and the [d-pad](../messages/dpad-update.md).

The navigation vocabulary is fixed to the six values above, so a host can safely treat anything else as unexpected. Keys are not sent as a finished string either: the controller starts from whatever text the host pre-filled, then streams the edits, so the host rebuilds the field by appending each inserted string and dropping the last character for every empty one.
