# Resources & Assets

Artwork travels inside the scheme document itself. There are two halves to it: `<Resource>` elements carry the actual image bytes, and `<Asset>` elements on each [display object](display-objects.md) point at those images by id. Splitting them this way lets several objects share one image without repeating it.

## Resources

A `<Resource>` is one embedded image. Its bytes are a PNG, Base64-encoded, wrapped in a `<data>` block inside the element. Resources are grouped together in a `<Resources>` block near the top of the scheme.

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | int | The resource's identifier, referenced by an asset's `resourceRef`. |
| `type` | string | The resource kind. Always `image` in practice. |

```xml
<Resources>
  <Resource id="1" type="image">
    <data><![CDATA[iVBORw0KGgoAAAANSUhEUgAAA... base64 PNG ...]]></data>
  </Resource>
  <Resource id="2" type="image">
    <data><![CDATA[iVBORw0KGgoAAAANSUhEUgAAAB... base64 PNG ...]]></data>
  </Resource>
</Resources>
```

The Base64 text sits in a `CDATA` section so the XML parser passes it through untouched. A controller reads everything inside `<data>`, decodes it, and keeps the resulting bitmap under the resource `id`. Because schemes are authored at a small design size, these images are usually low resolution, and a controller is free to downscale very large ones to fit its own memory limits before drawing.

## Assets

An `<Asset>` lives inside a [display object](display-objects.md) and binds one of that object's named image slots to a resource.

| Attribute | Type | Description |
|-----------|------|-------------|
| `name` | string | The slot this image fills (see the slot names below). |
| `resourceRef` | int | The `id` of the `<Resource>` to use for this slot. |

```xml
<DisplayObject id="3" type="button" hidden="no" top="0.1" left="0.1" width="0.2" height="0.3" functionHandler="fire">
  <Asset name="up" resourceRef="1" />
  <Asset name="down" resourceRef="2" />
</DisplayObject>
```

The slot names a display object expects depend on its `type`:

| Type | Asset slots |
|------|-------------|
| `button` | `up`, `down` |
| `image` | `up` |
| `dpad` | `left_up`, `up`, `right_up`, `left`, `inactive`, `right`, `left_down`, `down`, `right_down` |
| `text` | none (drawn from `text` / `textSize` / `color`) |

Several objects can point their assets at the same resource `id`, so a shared graphic only needs to be embedded once.
