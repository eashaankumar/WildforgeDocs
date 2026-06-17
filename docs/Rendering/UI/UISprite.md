# UISprite

Namespace:

```csharp
using Wildforge.Engine.Core.Rendering.UI;
```

`UISprite` renders a textured rectangle as part of a `UICanvas`.

It is the simplest built-in visual UI element and is commonly used for:

- panels
- icons
- backgrounds
- frames
- borders
- inventory slots
- decorative images

`UISprite` inherits from `UIElement`.

---

# Purpose

A `UISprite` draws a texture using a `RasterMaterial` and the element's `RectTransform`.

Unlike `UIButton`, a `UISprite` has no interaction logic.

It is purely visual.

---

# Inheritance

```text
UIElement

↓

UISprite
```

---

# Texture

```csharp
public Texture2D Texture;
```

The texture rendered by the sprite.

If no texture is assigned, nothing is drawn.

Example:

```csharp
sprite.Texture = iconTexture;
```

---

# Material

```csharp
public RasterMaterial Material;
```

The material used to render the sprite.

The material should support:

```text
MainTexture

Tint
```

`UISprite` automatically updates these material parameters before rendering.

---

# Color

```csharp
public Color Color = Color.White;
```

Tint color applied during rendering.

White leaves the texture unchanged.

Example:

```csharp
sprite.Color = Color.White;
```

Tinting:

```csharp
sprite.Color = Color.Red;
```

Transparent:

```csharp
sprite.Color = new Color(
    255,
    255,
    255,
    128);
```

---

# RectTransform

Like every `UIElement`, a sprite uses a `RectTransform`.

Example:

```csharp
sprite.RectTransform = new RectTransform
{
    Position = new Vector3(300, 150, 0),
    Size = new Vector2(128, 128),
    Scale = Vector3.One,
    Rotation = Quaternion.Identity,
    Pivot01 = new Vector2(0.5f, 0.5f)
};
```

The transform controls:

- position
- size
- scale
- rotation
- pivot

---

# SortingOrder

`UISprite` inherits:

```csharp
public int SortingOrder;
```

Example:

```text
Background

SortingOrder = 0
```

```text
Panel

SortingOrder = 10
```

```text
Icon

SortingOrder = 20
```

Higher sorting orders render later.

---

# Creating A Sprite

```csharp
var sprite = new UISprite
{
    Texture = panelTexture,
    Material = spriteMaterial,
    Color = Color.White,
    SortingOrder = 0,
    RectTransform = new RectTransform
    {
        Position = new Vector3(400, 300, 0),
        Size = new Vector2(300, 200),
        Scale = Vector3.One,
        Rotation = Quaternion.Identity,
        Pivot01 = new Vector2(0.5f, 0.5f)
    }
};

canvas.AddElement(sprite);
```

---

# Rendering

During rendering the sprite:

1. Assigns `Texture` to:

```text
MainTexture
```

2. Assigns `Color` to:

```text
Tint
```

3. Draws a textured quad using the element's `RectTransform`.

No additional work is required by the user.

---

# Using Multiple Sprites

A canvas may contain any number of sprites.

Example:

```text
Canvas

↓

Background

↓

Panel

↓

Icon

↓

Border
```

Each sprite renders independently.

---

# Layering

Sprites layer naturally using `SortingOrder`.

Example:

```csharp
background.SortingOrder = 0;

panel.SortingOrder = 10;

icon.SortingOrder = 20;
```

After changing sorting order:

```csharp
canvas.SortElements();
```

if the element has already been added to the canvas.

---

# Tinting

Tinting is useful for:

- disabled UI
- hover highlights
- flashing warnings
- health indicators
- team colors

Example:

```csharp
healthBar.Color = Color.Red;
```

or:

```csharp
inventorySlot.Color = Color.Gray;
```

No separate textures are required.

---

# Rotation

Sprites can be rotated using the `RectTransform`.

Example:

```csharp
sprite.RectTransform.Rotation =
    Quaternion.CreateFromAxisAngle(
        Vector3.UnitZ,
        MathF.PI * 0.5f);
```

Useful for:

- arrows
- indicators
- compass needles
- rotating icons

---

# Scaling

Sprites may be resized by changing:

```csharp
RectTransform.Size
```

or

```csharp
RectTransform.Scale
```

Changing size affects the UI rectangle.

Changing scale multiplies the final rendered size.

---

# Example: Background Panel

```csharp
var panel = new UISprite
{
    Texture = panelTexture,
    Material = uiMaterial,
    Color = Color.White,
    RectTransform = new RectTransform
    {
        Position = new Vector3(500, 300, 0),
        Size = new Vector2(500, 300),
        Scale = Vector3.One,
        Rotation = Quaternion.Identity,
        Pivot01 = new Vector2(0.5f, 0.5f)
    }
};

canvas.AddElement(panel);
```

---

# Example: Icon

```csharp
var icon = new UISprite
{
    Texture = swordIcon,
    Material = uiMaterial,
    Color = Color.White,
    SortingOrder = 100,
    RectTransform = new RectTransform
    {
        Position = new Vector3(64, 64, 0),
        Size = new Vector2(48, 48),
        Scale = Vector3.One,
        Rotation = Quaternion.Identity,
        Pivot01 = new Vector2(0.5f, 0.5f)
    }
};

canvas.AddElement(icon);
```

---

# Best Practices

## Do

Use sprites for decorative UI.

Reuse the same material where possible.

Use tinting instead of duplicate textures when only color changes.

Use `SortingOrder` to control layering.

---

## Don't

Do not use `UISprite` for interaction.

Do not create a new material for every sprite unless necessary.

Do not manually call `Render()`.

Do not forget to assign a texture.

---

# Relationship To Other Types

```text
UICanvas

↓

UISprite

↓

Texture2D

RasterMaterial

RectTransform
```

`UICanvas` owns the sprite.

`RectTransform` controls its placement.

`Texture2D` provides its image.

`RasterMaterial` defines how it is rendered.

---

# Summary

`UISprite` is Wildforge's basic image element.

It renders a textured rectangle using a `Texture2D`, a `RasterMaterial`, and a `RectTransform`, making it ideal for backgrounds, icons, decorative UI, and other non-interactive visual elements.