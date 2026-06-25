# Context Menu

A scheme can define a context menu: a short list of options the player reaches from the controller's own menu, separate from the on-screen controls. Games use it for things like resetting, toggling sound, or showing help. The menu is optional; a scheme without one simply has no extra options.

## Structure

The menu is a `<Menu>` block holding one `<Option>` per entry. It sits alongside the `<Resources>` and `<Layout>` blocks under the scheme root.

```xml
<Menu>
  <Option icon="1" title="Reset" event="reset" close="yes"/>
  <Option icon="2" title="Help" event="help" close="no"/>
</Menu>
```

## Option Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `title` | string | The label shown for the option. |
| `event` | string | The identifier reported to the game when the option is chosen. |
| `icon` | int | Selects one of the controller's built-in menu icons. |
| `close` | yes/no | Whether choosing the option dismisses the menu. |

## Behavior

When the player picks an option, the controller sends that option's `event` string back to the game as a `menuEvent` call, and the game decides what to do with it. The `close` flag is handled entirely on the controller: `yes` dismisses the menu after the choice, while `no` leaves it open so the player can pick again, which suits toggles like sound on and off.

`icon` is just an integer. Which picture it maps to is a controller convention rather than part of the protocol, so a controller supplies its own set of built-in icons and a game picks among them by index.
