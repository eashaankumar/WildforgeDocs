# SpriteSheet

`SpriteSheet` loads a texture atlas and exposes named sprites.

## Purpose

Store many sprites inside one PNG while referencing them by name.

## Loading

``` csharp
var sheet = SpriteSheet.Load(
    "terrain.png",
    "terrain.sheet",
    TextureImportSettings.PixelSRGB);
```

## Sheet format

``` text
grass 0 0 15 15
stone 16 0 31 15
```

Comments begin with `#`.

Coordinates are inclusive.

## Access

``` csharp
Texture2D grass = sheet["grass"].Texture;
```

Each entry exposes:

-   Id
-   Center
-   Size
-   Texture

## Design

-   Plain text metadata
-   Easy version control
-   Named lookup instead of indices
-   Cropped textures for simplicity
-   Atlas source retained for debugging

## Example

``` csharp
foreach(var sprite in sheet.Entries)
{
    Console.WriteLine(sprite.Key);
}
```
