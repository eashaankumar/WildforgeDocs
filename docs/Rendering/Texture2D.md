# Texture2D

`Texture2D` is the engine's CPU-side 2D image type.

It stores pixel data in memory, remembers how the texture should be imported and sampled, and is automatically uploaded to the GPU by the renderer when needed.

Use `Texture2D` for:

- Sprites
- UI images
- Font atlases
- Material textures
- Terrain textures
- Procedural textures
- Runtime-generated image data
- Data textures / lookup tables

---

## Basic Example

```csharp
var grass = Texture2D.Load(
    "Assets/Textures/grass.png",
    new TextureImportSettings
    {
        IsSRGB = true,
        Filter = TextureFilter.Trilinear,
        WrapMode = ImageWrapMode.Repeat
    });
```

For a sprite:

```csharp
var sprite = Texture2D.Load(
    "Assets/Sprites/player.png",
    TextureImportSettings.PixelSRGB);
```

For a data texture:

```csharp
var lookup = Texture2D.Load(
    "Assets/Data/lookup.png",
    TextureImportSettings.LinearData);
```

---

# Texture Import Flow

Texture import is controlled by `TextureImportSettings`.

```text
TextureImportSettings
        ↓
Texture2D
        ↓
ImageSampler
        ↓
Renderer
        ↓
Vulkan sampler + Vulkan image
```

The engine user specifies import behavior once, when creating/loading the texture.

The renderer does not need to be manually told how to sample the texture later.

---

# TextureImportSettings

`TextureImportSettings` describes how a texture should be imported.

```csharp
public readonly record struct TextureImportSettings(
    bool IsSRGB = true,
    TextureFilter Filter = TextureFilter.Bilinear,
    ImageWrapMode WrapMode = ImageWrapMode.Clamp,
    uint MipLevels = 0)
```

## Properties

| Property | Type | Description |
|---|---|---|
| `IsSRGB` | `bool` | Whether the texture should be treated as color data in sRGB space. |
| `Filter` | `TextureFilter` | User-facing filtering mode: `Pixel`, `Bilinear`, or `Trilinear`. |
| `WrapMode` | `ImageWrapMode` | What happens when UVs go outside the `0..1` range. |
| `MipLevels` | `uint` | Number of mip levels. `0` means generate the full mip chain when trilinear filtering is used. |

## Sampler

`TextureImportSettings` exposes an internal renderer-facing sampler:

```csharp
public ImageSampler Sampler => Filter switch
{
    TextureFilter.Pixel => new ImageSampler(
        ImageFilter.Nearest,
        ImageSampleMode.None,
        WrapMode,
        0f,
        0f),

    TextureFilter.Bilinear => new ImageSampler(
        ImageFilter.Linear,
        ImageSampleMode.None,
        WrapMode,
        0f,
        0f),

    TextureFilter.Trilinear => new ImageSampler(
        ImageFilter.Linear,
        ImageSampleMode.LinearMip,
        WrapMode),

    _ => throw new ArgumentOutOfRangeException(nameof(Filter), Filter, null)
};
```

Users usually do not need to create `ImageSampler` directly.

---

# TextureFilter

`TextureFilter` is the public filtering enum users choose from.

```csharp
public enum TextureFilter
{
    Pixel,
    Bilinear,
    Trilinear
}
```

## Pixel

```csharp
TextureFilter.Pixel
```

Uses nearest-neighbor filtering.

Use for:

- Pixel art
- Voxel-style textures
- Sharp sprites
- Low-resolution UI

Example:

```csharp
var texture = Texture2D.Load(
    "Assets/Sprites/player.png",
    new TextureImportSettings
    {
        IsSRGB = true,
        Filter = TextureFilter.Pixel,
        WrapMode = ImageWrapMode.Clamp
    });
```

## Bilinear

```csharp
TextureFilter.Bilinear
```

Uses smooth linear filtering without mipmaps.

Use for:

- UI images
- Icons
- Smooth sprites
- Font atlases
- Small textures that do not need mipmaps

Example:

```csharp
var texture = Texture2D.Load(
    "Assets/UI/button.png",
    new TextureImportSettings
    {
        IsSRGB = true,
        Filter = TextureFilter.Bilinear,
        WrapMode = ImageWrapMode.Clamp
    });
```

## Trilinear

```csharp
TextureFilter.Trilinear
```

Uses smooth filtering with mipmaps.

Use for:

- Terrain
- Large 3D textures
- Repeating surfaces
- Materials viewed at different distances

Example:

```csharp
var texture = Texture2D.Load(
    "Assets/Textures/stone_floor.png",
    new TextureImportSettings
    {
        IsSRGB = true,
        Filter = TextureFilter.Trilinear,
        WrapMode = ImageWrapMode.Repeat
    });
```

---

# ImageWrapMode

`ImageWrapMode` controls what happens when UV coordinates go outside the `0..1` range.

```csharp
public enum ImageWrapMode
{
    Clamp,
    Repeat,
    Mirror
}
```

## Clamp

```csharp
ImageWrapMode.Clamp
```

UVs outside the texture are clamped to the edge pixel.

Use for:

- UI
- Sprites
- Font atlases
- Non-repeating images

Example:

```csharp
var icon = Texture2D.Load(
    "Assets/UI/icon.png",
    new TextureImportSettings
    {
        IsSRGB = true,
        Filter = TextureFilter.Bilinear,
        WrapMode = ImageWrapMode.Clamp
    });
```

## Repeat

```csharp
ImageWrapMode.Repeat
```

The texture repeats when UVs go outside `0..1`.

Use for:

- Terrain
- Tiles
- Floors
- Walls
- Repeating material textures

Example:

```csharp
var grass = Texture2D.Load(
    "Assets/Textures/grass.png",
    new TextureImportSettings
    {
        IsSRGB = true,
        Filter = TextureFilter.Trilinear,
        WrapMode = ImageWrapMode.Repeat
    });
```

## Mirror

```csharp
ImageWrapMode.Mirror
```

The texture repeats, but every other repeat is mirrored.

Use for:

- Noise textures
- Pattern textures
- Reducing obvious seams

Example:

```csharp
var noise = Texture2D.Load(
    "Assets/Textures/noise.png",
    new TextureImportSettings
    {
        IsSRGB = false,
        Filter = TextureFilter.Bilinear,
        WrapMode = ImageWrapMode.Mirror
    });
```

---

# Presets

`TextureImportSettings` provides common presets.

## PixelSRGB

```csharp
TextureImportSettings.PixelSRGB
```

Equivalent to:

```csharp
new TextureImportSettings(
    IsSRGB: true,
    Filter: TextureFilter.Pixel,
    WrapMode: ImageWrapMode.Clamp)
```

Use for:

- Pixel art
- Sharp sprites
- Voxel textures

Example:

```csharp
var sprite = Texture2D.Load(
    "Assets/Sprites/player.png",
    TextureImportSettings.PixelSRGB);
```

## BilinearSRGB

```csharp
TextureImportSettings.BilinearSRGB
```

Equivalent to:

```csharp
new TextureImportSettings(
    IsSRGB: true,
    Filter: TextureFilter.Bilinear,
    WrapMode: ImageWrapMode.Clamp)
```

Use for:

- UI textures
- Smooth sprites
- Color images without mipmaps

Example:

```csharp
var button = Texture2D.Load(
    "Assets/UI/button.png",
    TextureImportSettings.BilinearSRGB);
```

## TrilinearSRGB

```csharp
TextureImportSettings.TrilinearSRGB
```

Equivalent to:

```csharp
new TextureImportSettings(
    IsSRGB: true,
    Filter: TextureFilter.Trilinear,
    WrapMode: ImageWrapMode.Clamp)
```

Use for:

- Material textures
- 3D model textures
- Terrain textures

If the texture should repeat, provide custom settings:

```csharp
var grass = Texture2D.Load(
    "Assets/Textures/grass.png",
    new TextureImportSettings
    {
        IsSRGB = true,
        Filter = TextureFilter.Trilinear,
        WrapMode = ImageWrapMode.Repeat
    });
```

## LinearData

```csharp
TextureImportSettings.LinearData
```

Equivalent to:

```csharp
new TextureImportSettings(
    IsSRGB: false,
    Filter: TextureFilter.Bilinear,
    WrapMode: ImageWrapMode.Clamp)
```

Use for:

- Font atlases
- Lookup tables
- Masks
- Normal maps
- Roughness maps
- Metallic maps
- Height maps
- Data textures

Example:

```csharp
var lookup = Texture2D.Load(
    "Assets/Data/lut.png",
    TextureImportSettings.LinearData);
```

---

# ImageSampler

`ImageSampler` is the renderer-facing sampler state.

```csharp
public readonly record struct ImageSampler(
    ImageFilter Filter,
    ImageSampleMode SampleMode,
    ImageWrapMode WrapMode,
    float MinLod = 0f,
    float MaxLod = 1000f)
```

Most user code should use `TextureImportSettings` instead of constructing `ImageSampler` directly.

## Properties

| Property | Type | Description |
|---|---|---|
| `Filter` | `ImageFilter` | Low-level texel filter: nearest or linear. |
| `SampleMode` | `ImageSampleMode` | Low-level mip sampling mode. |
| `WrapMode` | `ImageWrapMode` | Clamp, repeat, or mirror. |
| `MinLod` | `float` | Minimum mip LOD. |
| `MaxLod` | `float` | Maximum mip LOD. |

## Presets

```csharp
ImageSampler.PixelClamp
ImageSampler.BilinearClamp
ImageSampler.TrilinearClamp
ImageSampler.TrilinearRepeat

ImageSampler.Pixel
ImageSampler.Bilinear
ImageSampler.Trilinear
```

These are mainly useful internally or for advanced code.

---

# ImageFilter

`ImageFilter` is the low-level renderer-facing texel filter.

```csharp
public enum ImageFilter
{
    Nearest,
    Linear
}
```

| Value | Meaning |
|---|---|
| `Nearest` | Chooses the nearest texel. |
| `Linear` | Blends nearby texels. |

Public user code should usually use `TextureFilter` instead.

---

# ImageSampleMode

`ImageSampleMode` is the low-level renderer-facing mip sampling mode.

```csharp
public enum ImageSampleMode
{
    None,
    NearestMip,
    LinearMip
}
```

| Value | Meaning |
|---|---|
| `None` | No mip sampling. Uses only mip level `0`. |
| `NearestMip` | Chooses the nearest mip level. |
| `LinearMip` | Blends between adjacent mip levels. Used for trilinear filtering. |

Public user code should usually use `TextureFilter` instead.

---

# Texture2D

```csharp
public sealed class Texture2D : ISampledTexture2D
```

`Texture2D` stores CPU-side pixel data and texture metadata.

The renderer uploads it automatically when needed.

---

## Constructor

```csharp
public Texture2D(
    uint width,
    uint height,
    TextureImportSettings settings)
```

Creates an empty texture with the given size and import settings.

Example:

```csharp
var texture = new Texture2D(
    256,
    256,
    TextureImportSettings.BilinearSRGB);

texture.Fill(Color.White);
texture.Apply();
```

### Parameters

| Name | Type | Description |
|---|---|---|
| `width` | `uint` | Width in pixels. Must be greater than `0`. |
| `height` | `uint` | Height in pixels. Must be greater than `0`. |
| `settings` | `TextureImportSettings` | Import and sampling behavior. |

### Throws

| Exception | Condition |
|---|---|
| `ArgumentOutOfRangeException` | `width` or `height` is `0`. |

---

# Static Creation Methods

## Load

```csharp
public static Texture2D Load(
    string path,
    TextureImportSettings settings)
```

Loads an image file from disk.

The image is decoded as RGBA.

The texture is vertically flipped during import.

The texture is automatically applied before being returned.

Example:

```csharp
var texture = Texture2D.Load(
    "Assets/Textures/stone.png",
    new TextureImportSettings
    {
        IsSRGB = true,
        Filter = TextureFilter.Trilinear,
        WrapMode = ImageWrapMode.Repeat
    });
```

### Parameters

| Name | Type | Description |
|---|---|---|
| `path` | `string` | Path to the image file. |
| `settings` | `TextureImportSettings` | Import settings. |

### Returns

```csharp
Texture2D
```

A loaded texture.

---

## LoadFromBytes

```csharp
public static Texture2D LoadFromBytes(
    byte[] data,
    TextureImportSettings settings)
```

Loads an image from encoded image bytes.

Useful for:

- Embedded resources
- Network-loaded images
- Generated PNG/JPEG data
- Font atlas data

Example:

```csharp
var texture = Texture2D.LoadFromBytes(
    pngBytes,
    TextureImportSettings.LinearData);
```

### Parameters

| Name | Type | Description |
|---|---|---|
| `data` | `byte[]` | Encoded image bytes. |
| `settings` | `TextureImportSettings` | Import settings. |

### Returns

```csharp
Texture2D
```

A loaded texture.

---

## CalculateFullMipCount

```csharp
public static uint CalculateFullMipCount(
    uint width,
    uint height)
```

Calculates the full mip chain length for a texture.

Example:

```csharp
uint mipLevels = Texture2D.CalculateFullMipCount(1024, 1024);
```

A `1024 x 1024` texture produces:

```text
1024
512
256
128
64
32
16
8
4
2
1
```

So the mip count is `11`.

### Parameters

| Name | Type | Description |
|---|---|---|
| `width` | `uint` | Base mip width. |
| `height` | `uint` | Base mip height. |

### Returns

```csharp
uint
```

The number of mip levels needed to reach `1 x 1`.

---

# Properties

## Sampler

```csharp
public ImageSampler Sampler { get; set; }
```

The renderer-facing sampler used when the texture is bound to a shader.

This is normally assigned from `TextureImportSettings`.

Example:

```csharp
texture.Sampler = ImageSampler.TrilinearRepeat;
```

Most code should not mutate this after loading unless it intentionally wants to change how the texture is sampled.

---

## MipLevels

```csharp
public uint MipLevels { get; }
```

Number of mip levels allocated for this texture.

If the texture was imported with `TextureFilter.Trilinear`, this is either:

- The explicit `TextureImportSettings.MipLevels`
- The full mip chain if `MipLevels = 0`

If the texture is not trilinear, this is `1`.

---

## HasMipMaps

```csharp
public bool HasMipMaps => MipLevels > 1;
```

Returns whether the texture has more than one mip level.

Example:

```csharp
if (texture.HasMipMaps)
{
    // Texture uses mipmaps.
}
```

---

## Width

```csharp
public uint Width { get; }
```

Texture width in pixels.

---

## Height

```csharp
public uint Height { get; }
```

Texture height in pixels.

---

## AspectRatio

```csharp
public float AspectRatio => (float)Width / Height;
```

Width divided by height.

Useful for UI layout and sprite sizing.

---

## IsSRGB

```csharp
public readonly bool IsSRGB;
```

Whether the texture uses sRGB color conversion when uploaded to the GPU.

Use `true` for color textures.

Use `false` for data textures.

---

## Size

```csharp
public Vector2Int Size => new((int)Width, (int)Height);
```

Texture dimensions as a `Vector2Int`.

---

# Pixel Access

## GetPixel

```csharp
public Color GetPixel(
    uint x,
    uint y)
```

Gets one pixel.

Example:

```csharp
Color color = texture.GetPixel(10, 20);
```

### Throws

| Exception | Condition |
|---|---|
| `ArgumentOutOfRangeException` | `x >= Width` or `y >= Height`. |

---

## SetPixel

```csharp
public void SetPixel(
    uint x,
    uint y,
    Color color)
```

Sets one pixel and marks the texture dirty.

Example:

```csharp
texture.SetPixel(10, 20, Color.Red);
texture.Apply();
```

### Important

Call `Apply()` after editing pixels to increment the texture version and ensure the renderer uploads the latest data.

### Throws

| Exception | Condition |
|---|---|
| `ArgumentOutOfRangeException` | `x >= Width` or `y >= Height`. |

---

## GetPixels

```csharp
public Color[] GetPixels()
```

Returns a copy of all pixels.

Example:

```csharp
Color[] pixels = texture.GetPixels();
```

### Notes

This allocates a new array.

For non-allocating read access, use `GetPixelSpan()`.

---

## SetPixels

```csharp
public void SetPixels(
    Color[] pixels)
```

Copies a full pixel array into the texture and marks it dirty.

Example:

```csharp
var pixels = new Color[texture.Width * texture.Height];

for (var i = 0; i < pixels.Length; i++)
    pixels[i] = Color.White;

texture.SetPixels(pixels);
texture.Apply();
```

### Throws

| Exception | Condition |
|---|---|
| `ArgumentException` | Pixel array length does not match `Width * Height`. |

---

## GetPixelSpan

```csharp
public ReadOnlySpan<Color> GetPixelSpan()
```

Returns a read-only span over the internal pixel array.

Example:

```csharp
ReadOnlySpan<Color> pixels = texture.GetPixelSpan();

foreach (var pixel in pixels)
{
    // Read pixel.
}
```

### Notes

This does not allocate.

---

## GetMutablePixelSpan

```csharp
public Span<Color> GetMutablePixelSpan()
```

Returns a mutable span over the internal pixel array and marks the texture dirty.

Example:

```csharp
Span<Color> pixels = texture.GetMutablePixelSpan();

for (var i = 0; i < pixels.Length; i++)
    pixels[i] = Color.White;

texture.Apply();
```

### Notes

This is the fastest way to modify many pixels.

Always call `Apply()` after modifying the span.

---

# Raw RGBA Access

## GetRawRgba32

```csharp
public byte[] GetRawRgba32()
```

Returns texture pixels as RGBA8 bytes.

Each pixel uses four bytes:

```text
R G B A
```

Example:

```csharp
byte[] rgba = texture.GetRawRgba32();
```

### Notes

This allocates a new byte array.

The renderer uses this format for GPU upload.

---

## SetRawRgba32

```csharp
public void SetRawRgba32(
    ReadOnlySpan<byte> rgba)
```

Sets texture pixels from RGBA8 bytes and marks the texture dirty.

Example:

```csharp
texture.SetRawRgba32(rgbaBytes);
texture.Apply();
```

### Expected byte count

```csharp
Width * Height * 4
```

### Throws

| Exception | Condition |
|---|---|
| `ArgumentException` | Byte count does not equal `Width * Height * 4`. |

---

# Operations

## FlipY

```csharp
public void FlipY()
```

Flips the texture vertically and marks it dirty.

Example:

```csharp
texture.FlipY();
texture.Apply();
```

### Notes

`Texture2D.Load()` and `Texture2D.LoadFromBytes()` already call `FlipY()` during import.

You usually only call this manually for procedural or custom-imported pixel data.

---

## Fill

```csharp
public void Fill(
    Color color)
```

Fills the entire texture with a single color and marks it dirty.

Example:

```csharp
var texture = new Texture2D(
    128,
    128,
    TextureImportSettings.PixelSRGB);

texture.Fill(Color.Magenta);
texture.Apply();
```

Useful for:

- Placeholder textures
- White textures
- Error textures
- Debug textures
- Procedural generation

---

## Apply

```csharp
public void Apply()
```

Increments the texture version and marks the texture dirty.

The renderer uploads the new version the next time the texture is used.

Example:

```csharp
texture.SetPixel(0, 0, Color.Red);
texture.Apply();
```

### Important

Changing pixel data marks the texture dirty, but `Apply()` is what signals a new version.

Call `Apply()` after completing a batch of edits.

Good:

```csharp
for (uint y = 0; y < texture.Height; y++)
for (uint x = 0; x < texture.Width; x++)
{
    texture.SetPixel(x, y, Color.White);
}

texture.Apply();
```

Bad:

```csharp
for (uint y = 0; y < texture.Height; y++)
for (uint x = 0; x < texture.Width; x++)
{
    texture.SetPixel(x, y, Color.White);
    texture.Apply();
}
```

---

# Internal Methods

## MarkUploaded

```csharp
internal void MarkUploaded()
```

Called by the renderer after the GPU copy has completed.

User code should not call this.

---

# SpriteSheet Usage

`SpriteSheet` loads a source texture and crops it into individual `Texture2D` entries.

```csharp
var sheet = SpriteSheet.Load(
    "Assets/Sprites/characters.png",
    "Assets/Sprites/characters.sheet",
    TextureImportSettings.PixelSRGB);
```

Each entry contains:

```csharp
public sealed class SpriteSheetEntry
{
    public string Id { get; internal set; }
    public Vector2Int Center { get; }
    public Vector2Int Size { get; }
    public Texture2D Texture { get; }
}
```

## Accessing Sprites

```csharp
var idle = sheet["player_idle"];
Texture2D texture = idle.Texture;
```

## Sprite Sheet File Format

Each non-empty, non-comment line contains:

```text
id minX minY maxX maxY
```

Example:

```text
player_idle 0 0 31 31
player_run_0 32 0 63 31
player_run_1 64 0 95 31
```

Lines beginning with `#` are ignored.

```text
# id minX minY maxX maxY
player_idle 0 0 31 31
```

## Cropped Texture Behavior

Cropped textures inherit from the source texture:

- `IsSRGB`
- Filter behavior
- Wrap mode
- Mip settings

Internally, `SpriteSheet` reconstructs a matching `TextureImportSettings` from the source texture's sampler.

---

# Font Usage

Font assets load atlas textures using:

```csharp
TextureImportSettings.LinearData
```

This means:

```csharp
IsSRGB = false
Filter = TextureFilter.Bilinear
WrapMode = ImageWrapMode.Clamp
```

This is appropriate because font atlases are usually data-like coverage masks, not color photographs.

---

# Material Usage

`Texture2D` implements:

```csharp
ISampledTexture2D
```

so it can be used wherever the renderer expects a sampled texture.

Typical usage:

```csharp
var texture = Texture2D.Load(
    "Assets/Textures/grass.png",
    new TextureImportSettings
    {
        IsSRGB = true,
        Filter = TextureFilter.Trilinear,
        WrapMode = ImageWrapMode.Repeat
    });

material.SetTexture("BaseColor", texture);
```

The exact material method depends on the material API, but the important point is:

```text
Texture2D is the CPU/user-facing object.
The renderer resolves it into a GPU image and sampler.
```

---

# Renderer Behavior

The user does not manually upload textures.

The flow is:

```text
Texture2D created or loaded
        ↓
Texture used by material/render path
        ↓
VulkanTextureStore.EnsureUploaded(texture)
        ↓
GPU image created if needed
        ↓
GPU sampler created from texture.Sampler
        ↓
Texture pixels copied through staging buffer
        ↓
Mipmaps generated if needed
        ↓
Texture becomes shader-readable
```

When texture pixels change:

```text
SetPixel / SetPixels / SetRawRgba32 / Fill / FlipY
        ↓
Apply()
        ↓
Version increments
        ↓
Renderer sees new version
        ↓
Texture is re-uploaded
```

---

# Mipmaps

Mipmaps are enabled by choosing:

```csharp
TextureFilter.Trilinear
```

Example:

```csharp
var texture = Texture2D.Load(
    "Assets/Textures/terrain.png",
    new TextureImportSettings
    {
        IsSRGB = true,
        Filter = TextureFilter.Trilinear,
        WrapMode = ImageWrapMode.Repeat
    });
```

If `MipLevels` is `0`, the engine calculates the full mip chain.

```csharp
MipLevels = 0
```

means:

```text
Generate all mips down to 1x1.
```

For a `256 x 256` texture:

```text
256
128
64
32
16
8
4
2
1
```

For a `128 x 64` texture:

```text
128x64
64x32
32x16
16x8
8x4
4x2
2x1
1x1
```

## Explicit Mip Count

```csharp
var texture = Texture2D.Load(
    "Assets/Textures/terrain.png",
    new TextureImportSettings
    {
        IsSRGB = true,
        Filter = TextureFilter.Trilinear,
        WrapMode = ImageWrapMode.Repeat,
        MipLevels = 5
    });
```

This creates exactly five mip levels.

---

# Color Space

## Use sRGB for color textures

```csharp
IsSRGB = true
```

Use for:

- Base color
- Albedo
- Diffuse textures
- UI
- Sprites
- Icons

Example:

```csharp
var albedo = Texture2D.Load(
    "Assets/Textures/rock_albedo.png",
    new TextureImportSettings
    {
        IsSRGB = true,
        Filter = TextureFilter.Trilinear,
        WrapMode = ImageWrapMode.Repeat
    });
```

## Use linear for data textures

```csharp
IsSRGB = false
```

Use for:

- Normal maps
- Roughness maps
- Metallic maps
- Height maps
- AO maps
- Masks
- Lookup tables
- Font coverage masks

Example:

```csharp
var roughness = Texture2D.Load(
    "Assets/Textures/rock_roughness.png",
    new TextureImportSettings
    {
        IsSRGB = false,
        Filter = TextureFilter.Trilinear,
        WrapMode = ImageWrapMode.Repeat
    });
```

---

# Common Recipes

## Pixel Sprite

```csharp
var texture = Texture2D.Load(
    "Assets/Sprites/player.png",
    TextureImportSettings.PixelSRGB);
```

## Smooth UI Image

```csharp
var texture = Texture2D.Load(
    "Assets/UI/button.png",
    TextureImportSettings.BilinearSRGB);
```

## Repeating Terrain Texture

```csharp
var texture = Texture2D.Load(
    "Assets/Textures/grass.png",
    new TextureImportSettings
    {
        IsSRGB = true,
        Filter = TextureFilter.Trilinear,
        WrapMode = ImageWrapMode.Repeat
    });
```

## Normal Map

```csharp
var normal = Texture2D.Load(
    "Assets/Textures/rock_normal.png",
    new TextureImportSettings
    {
        IsSRGB = false,
        Filter = TextureFilter.Trilinear,
        WrapMode = ImageWrapMode.Repeat
    });
```

## Font Atlas

```csharp
var atlas = Texture2D.LoadFromBytes(
    atlasBytes,
    TextureImportSettings.LinearData);
```

## Procedural Solid Texture

```csharp
var texture = new Texture2D(
    64,
    64,
    TextureImportSettings.PixelSRGB);

texture.Fill(Color.White);
texture.Apply();
```

## Procedural Checkerboard

```csharp
var texture = new Texture2D(
    128,
    128,
    TextureImportSettings.PixelSRGB);

for (uint y = 0; y < texture.Height; y++)
for (uint x = 0; x < texture.Width; x++)
{
    var checker = ((x / 16) + (y / 16)) % 2 == 0;
    texture.SetPixel(x, y, checker ? Color.White : Color.Black);
}

texture.Apply();
```

## Fast Procedural Texture

```csharp
var texture = new Texture2D(
    512,
    512,
    TextureImportSettings.BilinearSRGB);

var pixels = texture.GetMutablePixelSpan();

for (var i = 0; i < pixels.Length; i++)
{
    pixels[i] = Color.White;
}

texture.Apply();
```

---

# Common Mistakes

## Forgetting Apply

Wrong:

```csharp
texture.SetPixel(0, 0, Color.Red);
```

Correct:

```csharp
texture.SetPixel(0, 0, Color.Red);
texture.Apply();
```

## Using sRGB for data textures

Wrong:

```csharp
var normal = Texture2D.Load(
    "normal.png",
    TextureImportSettings.TrilinearSRGB);
```

Correct:

```csharp
var normal = Texture2D.Load(
    "normal.png",
    new TextureImportSettings
    {
        IsSRGB = false,
        Filter = TextureFilter.Trilinear,
        WrapMode = ImageWrapMode.Repeat
    });
```

## Using Clamp for repeating terrain

Wrong:

```csharp
var grass = Texture2D.Load(
    "grass.png",
    TextureImportSettings.TrilinearSRGB);
```

Correct:

```csharp
var grass = Texture2D.Load(
    "grass.png",
    new TextureImportSettings
    {
        IsSRGB = true,
        Filter = TextureFilter.Trilinear,
        WrapMode = ImageWrapMode.Repeat
    });
```

## Using Trilinear for pixel art

Wrong:

```csharp
var sprite = Texture2D.Load(
    "player.png",
    TextureImportSettings.TrilinearSRGB);
```

Correct:

```csharp
var sprite = Texture2D.Load(
    "player.png",
    TextureImportSettings.PixelSRGB);
```

---

# Performance Notes

## Prefer batch edits

Good:

```csharp
var pixels = texture.GetMutablePixelSpan();

for (var i = 0; i < pixels.Length; i++)
    pixels[i] = Color.White;

texture.Apply();
```

Avoid calling `Apply()` repeatedly inside loops.

## Avoid unnecessary GetPixels calls

`GetPixels()` allocates a copy.

Prefer:

```csharp
ReadOnlySpan<Color> pixels = texture.GetPixelSpan();
```

for read-only access.

## Use SetRawRgba32 for raw import paths

If data is already RGBA8, use:

```csharp
texture.SetRawRgba32(rgbaBytes);
texture.Apply();
```

instead of setting pixels one at a time.

## Use mipmaps for large 3D textures

For textures seen at many distances, use:

```csharp
TextureFilter.Trilinear
```

This improves visual stability and reduces shimmering.

## Avoid mipmaps for UI and sprites

For UI and sprites, use:

```csharp
TextureFilter.Pixel
```

or:

```csharp
TextureFilter.Bilinear
```

Mipmaps are usually unnecessary for screen-space images.

---

# API Summary

## Enums

```csharp
public enum TextureFilter
{
    Pixel,
    Bilinear,
    Trilinear
}
```

```csharp
public enum ImageWrapMode
{
    Clamp,
    Repeat,
    Mirror
}
```

```csharp
public enum ImageFilter
{
    Nearest,
    Linear
}
```

```csharp
public enum ImageSampleMode
{
    None,
    NearestMip,
    LinearMip
}
```

## Structs

```csharp
public readonly record struct TextureImportSettings(
    bool IsSRGB = true,
    TextureFilter Filter = TextureFilter.Bilinear,
    ImageWrapMode WrapMode = ImageWrapMode.Clamp,
    uint MipLevels = 0)
```

```csharp
public readonly record struct ImageSampler(
    ImageFilter Filter,
    ImageSampleMode SampleMode,
    ImageWrapMode WrapMode,
    float MinLod = 0f,
    float MaxLod = 1000f)
```

## Texture2D Methods

```csharp
public Texture2D(
    uint width,
    uint height,
    TextureImportSettings settings)
```

```csharp
public static Texture2D Load(
    string path,
    TextureImportSettings settings)
```

```csharp
public static Texture2D LoadFromBytes(
    byte[] data,
    TextureImportSettings settings)
```

```csharp
public static uint CalculateFullMipCount(
    uint width,
    uint height)
```

```csharp
public void FlipY()
```

```csharp
public Color GetPixel(
    uint x,
    uint y)
```

```csharp
public void SetPixel(
    uint x,
    uint y,
    Color color)
```

```csharp
public Color[] GetPixels()
```

```csharp
public void SetPixels(
    Color[] pixels)
```

```csharp
public ReadOnlySpan<Color> GetPixelSpan()
```

```csharp
public Span<Color> GetMutablePixelSpan()
```

```csharp
public byte[] GetRawRgba32()
```

```csharp
public void SetRawRgba32(
    ReadOnlySpan<byte> rgba)
```

```csharp
public void Fill(
    Color color)
```

```csharp
public void Apply()
```

---

# Recommended Defaults

| Use Case | Settings |
|---|---|
| Pixel art sprite | `TextureImportSettings.PixelSRGB` |
| Smooth UI | `TextureImportSettings.BilinearSRGB` |
| Font atlas | `TextureImportSettings.LinearData` |
| Albedo material texture | `IsSRGB = true`, `Filter = Trilinear`, `WrapMode = Repeat` |
| Normal map | `IsSRGB = false`, `Filter = Trilinear`, `WrapMode = Repeat` |
| Roughness/metallic/AO | `IsSRGB = false`, `Filter = Trilinear`, `WrapMode = Repeat` |
| Lookup/data texture | `TextureImportSettings.LinearData` |
| Terrain | `IsSRGB = true`, `Filter = Trilinear`, `WrapMode = Repeat` |

---

# Design Rule

User-facing texture import should be configured through:

```csharp
TextureImportSettings
```

Renderer-facing sampling state should be represented by:

```csharp
ImageSampler
```

The user should not have to know that:

```text
Bilinear = Linear filter + no mip interpolation
Trilinear = Linear filter + linear mip interpolation
```

The public API exposes:

```csharp
TextureFilter.Pixel
TextureFilter.Bilinear
TextureFilter.Trilinear
```

The renderer converts those into the lower-level sampler state internally.