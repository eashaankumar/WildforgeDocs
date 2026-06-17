# UIText

Namespace:

```csharp
using Wildforge.Engine.Core.Rendering.UI;
```

`UIText` renders text inside a user interface.

It is used for:

- labels
- buttons
- menus
- HUD elements
- inventory text
- health values
- dialogue
- debug overlays

`UIText` inherits from `UIElement`.

---

# Purpose

A `UIText` displays text using a `FontAsset`.

Unlike `UISprite`, which renders images, `UIText` renders glyphs while supporting:

- horizontal alignment
- vertical alignment
- font size
- color
- arbitrary transforms

---

# Inheritance

```text
UIElement

↓

UIText
```

---

# FontAsset

```csharp
public FontAsset FontAsset;
```

The font used to render the text.

A valid font must be assigned.

Example:

```csharp
label.FontAsset = font;
```

If no font is assigned, nothing is rendered.

---

# Text

```csharp
public string Text;
```

The string to display.

Example:

```csharp
label.Text = "Play";
```

Empty strings are ignored during rendering.

---

# FontSize

```csharp
public float FontSize;
```

Controls the size of the rendered text.

Example:

```csharp
label.FontSize = 24;
```

Larger values produce larger glyphs.

---

# Color

```csharp
public Color Color;
```

The color applied to every glyph.

Example:

```csharp
label.Color = Color.White;
```

Colored text:

```csharp
label.Color = Color.Red;
```

Transparent text:

```csharp
label.Color = new Color(
    255,
    255,
    255,
    128);
```

---

# Horizontal Alignment

```csharp
public TextXAlignment XAlign;
```

Possible values:

```csharp
public enum TextXAlignment
{
    Left,
    Center,
    Right
}
```

### Left

Text begins at the left side of the element.

### Center

Text is centered horizontally.

### Right

Text is aligned to the right side.

Example:

```csharp
label.XAlign =
    UIText.TextXAlignment.Center;
```

---

# Vertical Alignment

```csharp
public TextYAlignment YAlign;
```

Possible values:

```csharp
public enum TextYAlignment
{
    Top,
    Center,
    Bottom
}
```

### Top

Text is aligned to the top.

### Center

Text is vertically centered.

### Bottom

Text begins at the bottom.

Example:

```csharp
label.YAlign =
    UIText.TextYAlignment.Center;
```

---

# RectTransform

Like every UI element, text uses a `RectTransform`.

Example:

```csharp
label.RectTransform = new RectTransform
{
    Position = new Vector3(400, 300, 0),
    Size = new Vector2(300, 80),
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

# Automatic Alignment

Before rendering, `UIText` measures the text using the assigned `FontAsset`.

It then automatically computes the correct position based on:

- element size
- horizontal alignment
- vertical alignment
- pivot

No manual positioning of glyphs is required.

---

# Creating Text

```csharp
var title = new UIText
{
    FontAsset = font,
    Text = "Wildforge",
    FontSize = 42,
    Color = Color.White,

    XAlign = UIText.TextXAlignment.Center,
    YAlign = UIText.TextYAlignment.Center,

    RectTransform = new RectTransform
    {
        Position = new Vector3(400, 120, 0),
        Size = new Vector2(600, 80),
        Scale = Vector3.One,
        Rotation = Quaternion.Identity,
        Pivot01 = new Vector2(0.5f, 0.5f)
    }
};

canvas.AddElement(title);
```

---

# Rendering

During rendering, `UIText`:

1. Measures the text bounds.
2. Computes horizontal alignment.
3. Computes vertical alignment.
4. Applies the element's transform.
5. Draws the text.

Rendering is skipped when:

- `FontAsset` is `null`
- `Text` is empty

---

# Rotation

Text can be rotated using its `RectTransform`.

Example:

```csharp
label.RectTransform.Rotation =
    Quaternion.CreateFromAxisAngle(
        Vector3.UnitZ,
        MathF.PI / 4);
```

Useful for:

- indicators
- stylized menus
- rotated HUD elements

---

# Scaling

Changing:

```csharp
RectTransform.Scale
```

scales the rendered text.

Changing:

```csharp
FontSize
```

changes the glyph size itself.

These can be combined.

---

# Example: HUD Label

```csharp
var score = new UIText
{
    FontAsset = font,
    Text = "Score: 1250",
    FontSize = 20,
    Color = Color.White,

    XAlign = UIText.TextXAlignment.Left,
    YAlign = UIText.TextYAlignment.Top,

    RectTransform = new RectTransform
    {
        Position = new Vector3(20, 700, 0),
        Size = new Vector2(300, 40),
        Scale = Vector3.One,
        Rotation = Quaternion.Identity,
        Pivot01 = Vector2.Zero
    }
};

canvas.AddElement(score);
```

---

# Example: Centered Title

```csharp
var title = new UIText
{
    FontAsset = font,
    Text = "Main Menu",
    FontSize = 48,
    Color = Color.Yellow,

    XAlign = UIText.TextXAlignment.Center,
    YAlign = UIText.TextYAlignment.Center
};
```

The text automatically appears centered inside its rectangle.

---

# Dynamic Text

The displayed string can be changed at any time.

Example:

```csharp
fpsLabel.Text = $"FPS: {fps}";
```

On the next frame the new text is rendered automatically.

---

# Best Practices

## Do

Assign a valid `FontAsset`.

Choose alignment instead of manually offsetting text.

Reuse font assets across multiple labels.

Update the `Text` property directly for dynamic UI.

---

## Don't

Do not render text without a font.

Do not manually compute glyph positions.

Do not create duplicate font assets unnecessarily.

Do not call `Render()` yourself.

---

# Relationship To Other Types

```text
UICanvas

↓

UIText

↓

FontAsset

RectTransform

Color
```

`UICanvas` owns the text element.

`FontAsset` provides glyph data.

`RectTransform` controls placement.

`Color` controls the rendered glyph color.

---

# Summary

`UIText` is Wildforge's built-in text rendering element.

It automatically measures, aligns, and renders text inside a `RectTransform`, making it suitable for labels, menus, HUD elements, buttons, and other user interface text.