# Texture2D

Represents a CPU-side 2D texture that is automatically uploaded to the GPU when first used.

## Loading a Texture

```csharp
var texture = Texture2D.Load(
    "Assets/Textures/Grass.png",
    new TextureImportSettings
    {
        IsSRGB = true,
        Filter = TextureFilter.Trilinear,
        WrapMode = ImageWrapMode.Repeat
    });
```

## TextureImportSettings

| Property | Description |
|----------|-------------|
| `IsSRGB` | Imports the texture as sRGB. |
| `Filter` | Controls texture filtering. |
| `WrapMode` | Controls UV wrapping outside the 0-1 range. |
| `MipLevels` | Number of mip levels to generate. `0` generates the full mip chain. |

## TextureFilter

### Pixel

Uses nearest-neighbour filtering.

### Bilinear

Uses bilinear filtering.

### Trilinear

Uses trilinear filtering with mipmaps.

## ImageWrapMode

### Clamp

Clamps UVs to the texture edge.

### Repeat

Repeats the texture.

### Mirror

Repeats while mirroring every other tile.