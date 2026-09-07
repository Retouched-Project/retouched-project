# Display Objects

A `<DisplayObject>` is one item in the layout: a button, a d-pad, a static image, or a piece of text. Every object shares a common set of attributes, and a few attributes only apply to certain types. The `type` attribute decides which.

## Common Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | int | Identifier for the object within the scheme. Not used to match objects across [updates](xml-schema.md#updates), which replace the layout wholesale, and some writers omit it entirely. |
| `type` | string | One of `button`, `image`, `text`, or `dpad`. |
| `left` | float | Left edge, normalized `0` to `1`. |
| `top` | float | Top edge, normalized `0` to `1`. |
| `width` | float | Width, normalized `0` to `1`. |
| `height` | float | Height, normalized `0` to `1`. |
| `hidden` | yes/no | When `yes`, the object is neither drawn nor interactive. |
| `functionHandler` | string | The identifier the controller reports when the object is used. |
| `sample` | string | Optional per-object image sampling, `linear` or `nearest`. Overrides the scheme default. |

Coordinates follow the [normalized system](xml-schema.md#coordinate-system): they are fractions of the scheme's `width` and `height`, not pixels.

`sample` is inherited. An object starts with the scheme's `sample` value and only carries its own `sample` attribute when it differs, so an absent `sample` means "use the scheme default."

An authoring tool may also emit a `name` attribute. It is layout metadata and controllers ignore it.

## button

An interactive button with a pressed and unpressed appearance. When the user presses or releases it, the controller reports the button's `functionHandler` so the game knows which control was used.

A button references two images by [asset](resources.md) name:

| Asset name | Use |
|------------|-----|
| `up` | Drawn while the button is idle. |
| `down` | Drawn while the button is held. |

```xml
<DisplayObject id="2" type="button" hidden="no" top="0.1" left="0.1" width="0.2" height="0.3" sample="linear" functionHandler="fire">
  <Asset name="up" resourceRef="1" />
  <Asset name="down" resourceRef="2" />
</DisplayObject>
```

A button may also carry a [`<HitRect>`](#hit-rect) to make its touch area differ from its drawn area.

## image

A static, non-interactive picture, such as a background or a label. It draws a single `up` [asset](resources.md) and reports no input.

```xml
<DisplayObject id="3" type="image" hidden="no" top="0" left="0" width="1" height="1" functionHandler="up">
  <Asset name="up" resourceRef="4" />
</DisplayObject>
```

## text

A piece of text drawn directly by the controller, with no artwork. It uses three extra attributes instead of assets.

| Attribute | Type | Description |
|-----------|------|-------------|
| `text` | string | The string to display. |
| `textSize` | float | Text height, normalized as a fraction of the scheme `height`. |
| `color` | hex | Text color. Six hex digits (`RRGGBB`, fully opaque) or eight (`AARRGGBB`, with alpha). |

```xml
<DisplayObject id="5" type="text" hidden="no" top="0.05" left="0.4" width="0.2" height="0.08" text="P1" textSize="0.06" color="ffffff"/>
```

## dpad

A directional pad. Instead of a single press it reports a direction, and it carries two tuning attributes.

| Attribute | Type | Description |
|-----------|------|-------------|
| `deadzone` | float | Size of the dead area at the center where no direction registers, as a fraction `0` to `1` of the pad. Defaults to `0.25` when absent; values above `1` are ignored. |
| `radial` | yes/no | When `yes`, the pad reads touches radially by angle and distance from the center, using a circular dead zone. When `no`, it uses a rectangular center dead zone. Defaults to `yes` when absent, and no endpoint writes it, so hand authored documents are the only place it appears. |

A d-pad draws up to nine [asset](resources.md) frames, one per state:

| Asset names |
|-------------|
| `left_up`, `up`, `right_up` |
| `left`, `inactive`, `right` |
| `left_down`, `down`, `right_down` |

`inactive` is the resting frame; the others are shown for the matching direction. See [d-pad updates](../messages/dpad-update.md) for what the pad reports.

```xml
<DisplayObject id="6" name="dpad" type="dpad" hidden="no" deadzone="0.25" top="0.55" left="0.05" width="0.35" height="0.6">
  <Asset name="up" resourceRef="7" />
  <Asset name="down" resourceRef="8" />
  <Asset name="left" resourceRef="9" />
  <Asset name="right" resourceRef="10" />
  <Asset name="inactive" resourceRef="11" />
</DisplayObject>
```

## Hit Rect

An interactive object (a button or a d-pad) may include a single `<HitRect>` child to define the area that responds to touch, separately from the area it draws into.

| Attribute | Type | Description |
|-----------|------|-------------|
| `left` | float | Left edge of the hit area, normalized `0` to `1`. |
| `top` | float | Top edge, normalized `0` to `1`. |
| `width` | float | Width, normalized `0` to `1`. |
| `height` | float | Height, normalized `0` to `1`. |

```xml
<HitRect left="0.08" top="0.45" width="0.3" height="0.3" />
```

When a `<HitRect>` is absent, the hit area is simply the object's own `left`/`top`/`width`/`height` bounds.

On a d-pad it does one more job. A pad follows the finger that is dragging it, and the hit rect is the region it may be dragged within, so a d-pad whose hit rect is larger than its drawn bounds can float around inside it. This is the usual reason a d-pad carries one, and it means a hit rect that merely matches the drawn bounds pins the pad in place.
