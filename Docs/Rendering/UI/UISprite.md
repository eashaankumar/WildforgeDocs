# UI Sprite

Create 2D sprites in screen space. UI Camera recommended.
But can be rendered in 3D.

* Can be rotated and scaled
* Custom shaders and materials
* Texture2D support
* 3D position (including Z)

## Usage

Creation:

```csharp
var spriteTexture2D = Texture2D.Load("...")

var _sprite = new UISprite
{
    material = new RasterMaterial(spritePipeline),
    texture = spriteTexture2D,
    rectTransform = new RectTransform
    {
        Position = new Vector3(_window.Width / 2, _window.Height - 75, 0),
        Rotation = Quaternion.Identity,
        Size = new Vector2(
            spriteTexture2D.Width * 0.5f,
            spriteTexture2D.Height * 0.5f),
        Pivot01 = new Vector2(0.5f, 1f),
        Scale = Vector3.One
    }
};
```

Rendering:

```csharp
var uiPass = _renderer.CreateRasterPass(_uiCamera, rasterColor, rasterDepth);
uiPass.DrawSprite(_sprite);

```