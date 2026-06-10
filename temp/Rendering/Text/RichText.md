# Rich Text Rendering

Forgewild supports rich text through the `RichText` class. Rich text lets individual glyphs override rendering properties such as position offset, rotation offset, scale multiplier, color, boldness, and italic skew.

Regular string text rendering does **not** parse markup. Use `RichText` when markup is needed.

---

## Basic Usage

```csharp
private RichText _titleText = null!;
```

Create or parse rich text once, usually in `OnLoad()`:

```csharp
_titleText = RichText.Parse(
    "<Style color={1,0,0,1} bold={0.05}>H</Style>ello World!");
```

Draw it every frame:

```csharp
_renderer.DrawText(
    _titleText,
    new TextStyle(
        _font,
        50.0f,
        Color.White.ToVector4()),
    Mathf.TRS(new Vector3(0, 0, 10), Quaternion.Identity, Vector3.One));
```

`RichText` caches its parsed glyph data. It reparses only when `Source` changes.

---

## Plain Text vs Rich Text

Use plain strings for normal text:

```csharp
_renderer.DrawText(
    "Hello World!",
    new TextStyle(_font, 50.0f, Color.White.ToVector4()),
    transform);
```

This path does not parse rich text tags.

Use `RichText` for styled glyphs:

```csharp
var text = RichText.Parse(
    "<Style color={1,0,0,1}>H</Style>ello World!");

_renderer.DrawText(text, style, transform);
```

---

## Supported Markup

Rich text uses `<Style ...>` and `</Style>` tags.

Example:

```xml
<Style posOff={0.1,2,3} rotOff={45,20,30} scaleMult={0.81} color={0.1,0.2,1,1} bold={0.05} italic={0.1}>H</Style>ello World!
```

Only the text inside the style tag receives the override.

---

## Style Attributes

### `posOff`

Adds a per-glyph local-space position offset.

```xml
<Style posOff={0,1,0}>Jump</Style>
```

Format:

```text
posOff={x,y,z}
```

---

### `rotOff`

Adds a per-glyph rotation offset in degrees.

```xml
<Style rotOff={0,0,45}>Tilted</Style>
```

Format:

```text
rotOff={xDegrees,yDegrees,zDegrees}
```

---

### `scaleMult`

Multiplies the rendered glyph scale without changing layout advance.

```xml
<Style scaleMult={1.5}>Big</Style>
```

Format:

```text
scaleMult={multiplier}
```

---

### `color`

Overrides glyph color.

```xml
<Style color={1,0,0,1}>Red</Style>
```

Format:

```text
color={r,g,b,a}
```

Color values are linear float values.

---

### `bold`

Adjusts MSDF thickness.

```xml
<Style bold={0.05}>Bold</Style>
```

Recommended values are small, usually:

```text
0.02 - 0.10
```

Very large values can destroy the MSDF edge.

---

### `italic`

Applies glyph skew.

```xml
<Style italic={0.15}>Italic</Style>
```

Format:

```text
italic={skewAmount}
```

---

## Updating Rich Text

To change the source text:

```csharp
_titleText.Source =
    "<Style color={0,1,0,1}>N</Style>ew text";
```

Changing `Source` marks the rich text cache dirty. The next draw reparses it once.

You can also force parsing immediately:

```csharp
_titleText.Reparse();
```

---

## Performance Notes

Do not create or parse rich text every frame.

Good:

```csharp
// OnLoad
_titleText = RichText.Parse("<Style color={1,0,0,1}>H</Style>ello");

// Render
_renderer.DrawText(_titleText, style, transform);
```

Bad:

```csharp
// Avoid doing this every frame
_renderer.DrawText(
    RichText.Parse("<Style color={1,0,0,1}>H</Style>ello"),
    style,
    transform);
```

Parsing allocates and should only happen when the source string changes.

---

## Layout Behavior

Rich text style effects are render-only by default.

This means:

```xml
<Style scaleMult={2}>H</Style>ello
```

renders a larger `H`, but the rest of the text is still laid out as if `H` had normal advance.

This is useful for animation and glyph effects. If layout-affecting styling is needed later, add a separate property such as:

```text
layoutScale={2}
```

---

## Current Limitations

- Only `<Style ...>` tags are supported.
- Nested style tags are not fully supported unless the parser is extended with a style stack.
- Rich text currently affects glyph rendering, not text layout advance.
- Multiple font atlases in one frame still require batching changes in `TextRenderer`.

---

## Recommended Usage Pattern

```csharp
private FontAsset _font = null!;
private RichText _richText = null!;

public override void OnLoad()
{
    _font = FontAsset.Load(Path.Combine(
        EnginePaths.AssetRoot,
        "Fonts",
        "pixeloid-font",
        "PixeloidSans.font"));

    _richText = RichText.Parse(
        "<Style color={1,0,0,1} bold={0.05}>H</Style>ello World!");
}

public override void Render()
{
    _renderer.DrawText(
        _richText,
        new TextStyle(
            _font,
            50.0f,
            Color.White.ToVector4()),
        Mathf.TRS(new Vector3(0, 0, 10), Quaternion.Identity, Vector3.One));
}
```
