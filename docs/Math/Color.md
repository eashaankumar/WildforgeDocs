# Color

Namespace:

``` csharp
using Wildforge.Engine.Core.Maths;
```

`Color` represents an RGBA color using normalized floating-point
channels.

``` csharp
public struct Color
{
    public float R;
    public float G;
    public float B;
    public float A;
}
```

## Purpose

`Color` is the standard color representation used throughout Wildforge
for rendering, UI, textures, procedural generation, material colors and
interpolation.

Channels are normalized to the range **0.0--1.0**.

## Creating Colors

``` csharp
var red = new Color(1f, 0f, 0f, 1f);
```

The constructor clamps all channels into the valid range.

### From RGB

``` csharp
Color c = Color.FromRgb(1f, 0.5f, 0f);
```

### From Color32

``` csharp
Color c = Color.From255(new Color32(255,128,0,255));
```

### From Hex

Supported formats:

-   `#RGB`
-   `#RGBA`
-   `#RRGGBB`
-   `#RRGGBBAA`

``` csharp
Color gold = Color.FromHex("#ffcc00");
```

## Conversions

``` csharp
Vector3 rgb = color.ToVector3();
Vector4 rgba = color.ToVector4();
```

## HSV

``` csharp
HsvColor hsv = color.ToHsv();
Color rgb = hsv.ToRgb();
```

Useful for hue shifting, palette generation, biome variation and UI
themes.

## Interpolation

``` csharp
Color.Lerp(a, b, t);
Color.LerpAlpha(a, alpha, t);
Color.LerpHSVHue(a, hue, t);
Color.LerpHSVSaturation(a, saturation, t);
Color.LerpHSVValue(a, value, t);
```

## Built-in Colors

Wildforge exposes many predefined colors including:

-   Black
-   White
-   Clear
-   Gray variants
-   Rainbow colors
-   Success / Warning / Error
-   Grass / Water / Dirt / Sand
-   Gold / Silver / Copper
-   Sky colors

## Color32

`Color32` stores RGBA as bytes.

Use it for image import/export and compact storage.

## Design

-   Normalized floats are renderer-friendly.
-   Constructor clamps invalid values.
-   Public fields minimize overhead.
-   HSV support enables procedural workflows.

## Example

``` csharp
Color tint = Color.FromHex("#44cc88");

tint = Color.LerpHSVHue(
    tint,
    180f,
    0.5f);
```
