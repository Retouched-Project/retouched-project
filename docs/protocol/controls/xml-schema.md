# Control Scheme XML

A control scheme is the on-screen layout a game hands to the controller: the buttons, d-pad, images, and text that make up the gamepad, along with the artwork to draw them with. The game delivers it as a single XML document (see [RequestXML & Delivery](../session/request-xml.md)), the controller parses it, and from then on the controller renders that layout and reports the matching input back.

## Document Structure

The document is a `<BMApplicationScheme>` root that carries the scheme-wide settings as attributes and holds three kinds of child content: an optional context [`<Menu>`](context-menu.md), a [`<Resources>`](resources.md) block of artwork, and a `<Layout>` block of [display objects](display-objects.md).

```mermaid
graph TD
    S["BMApplicationScheme"] --> M["Menu (optional)"]
    S --> R["Resources"]
    S --> L["Layout"]
    M --> O["Option"]
    R --> RES["Resource"]
    RES --> D["data (Base64 PNG)"]
    L --> DO["DisplayObject"]
    DO --> A["Asset"]
    DO --> H["HitRect (optional)"]
```

A concrete document then looks like this:

```xml
<?xml version="1.0" encoding="utf-8"?>
<BMApplicationScheme version="0.1" orientation="landscape" touchEnabled="yes" width="480" height="320" sample="linear" accelerometerEnabled="yes">
  <Resources>
    <Resource id="1" type="image">
      <data><![CDATA[iVBORw0KGgoAAAANSUhEUg...]]></data>
    </Resource>
  </Resources>
  <Layout>
    <DisplayObject id="2" type="button" hidden="no" top="0.1" left="0.1" width="0.2" height="0.3" functionHandler="fire">
      <Asset name="up" resourceRef="1" />
      <Asset name="down" resourceRef="1" />
    </DisplayObject>
  </Layout>
</BMApplicationScheme>
```

## Root Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `version` | string | Scheme format version. Always `0.1` in practice. |
| `orientation` | string | `landscape` or `portrait`. The orientation the controller should present. |
| `width` | int | Design width of the layout, in scheme units. Typically `480` (landscape) or `320` (portrait). |
| `height` | int | Design height of the layout, in scheme units. Typically `320` (landscape) or `480` (portrait). |
| `touchEnabled` | yes/no | Whether the scheme accepts raw [touch](../messages/touch.md) input. |
| `accelerometerEnabled` | yes/no | Whether the scheme uses the [accelerometer](../messages/acceleration.md). |
| `sample` | string | Default image sampling for the whole scheme, `linear` or `nearest`. Individual objects can override it (see [Display Objects](display-objects.md)). |

Every `yes/no` attribute follows the same rule: any value other than the exact string `no` counts as yes.

## Coordinate System

Positions and sizes are not pixels. Every `left`, `top`, `width`, and `height` (on both display objects and their hit rects) is a normalized fraction in the range `0` to `1`, measured against the scheme's `width` and `height`. A button at `left="0.5"` sits halfway across the layout regardless of the device's real resolution. The controller scales those fractions to its own screen, which is what lets one scheme fit any phone.

## Parsing

Controllers parse the document by element name as it streams in, rather than by strict nesting. The meaningful elements are the leaves: `BMApplicationScheme`, `Resource`, `DisplayObject`, `Asset`, `HitRect`, `Option`, and the `data` block inside a resource. The `<Resources>`, `<Layout>`, and `<Menu>` wrappers are just containers and carry no attributes of their own, so a parser can treat them as grouping and act only on the leaves it recognizes, ignoring anything it does not.

A game can also send a scheme again mid-session to change the layout (for example when a menu opens). An update is just another scheme document; the controller applies it over the current one, replacing the display objects it contains.

## See Also

- [Display Objects](display-objects.md) for the `<DisplayObject>` element and its types.
- [Resources & Assets](resources.md) for how artwork is embedded and referenced.
- [Context Menu](context-menu.md) for the `<Menu>` and its options.
- [RequestXML & Delivery](../session/request-xml.md) for how the document reaches the controller.
