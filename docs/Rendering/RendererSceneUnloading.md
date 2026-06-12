# Renderer Scene Unloading

`VulkanRenderer.Unload(Action disposeResources)` provides the correct and safe sequence for releasing all renderer-owned resources used by a scene.

## Purpose

GPU resources may still be referenced by in-flight command buffers and cached descriptor sets when a scene is unloaded. Calling `Unload()` guarantees that these resources are released safely and that any renderer caches referencing them are invalidated.

## Usage

Dispose all renderer-owned resources inside the callback.

```csharp
public override void OnUnload()
{
    _renderer.Unload(() =>
    {
        _framebuffers.Dispose();

        _triangleInstanceBuffer.Dispose();
        _indirectArgsBuffer.Dispose();

        _compute.Dispose();

        _copyMaterial.Dispose();

        _redRasterMaterial.Dispose();
        _blueRasterMaterial.Dispose();
        _indirectRasterMaterial.Dispose();

        _redRtMaterial.Dispose();
        _blueRtMaterial.Dispose();

        _renderer.Meshes.Destroy(ref _triangleMesh);
    });

    // Dispose non-renderer resources here if needed.
}
```

---

## Unload Sequence

Internally, the renderer performs the following sequence:

```text
WaitIdle()
    ↓
disposeResources()
    ↓
ClearScenePersistentCaches()
```

This ensures:

- The GPU has finished using all scene resources.
- Renderer-owned resources are safely destroyed.
- Cached descriptors and material buffers referencing those resources are cleared.

---

## Renderer Resources

Dispose any resources created by the renderer, including:

- `RenderTexture`
- `RasterMaterial`
- `RTMaterial`
- `GraphicsBuffer`
- `ComputeShader`
- Scene-created meshes
- Raytracing acceleration structures and instances
- Any other renderer-owned GPU resource

Example:

```csharp
_renderer.Unload(() =>
{
    _renderTexture.Dispose();
    _material.Dispose();
    _buffer.Dispose();

    _renderer.Meshes.Destroy(ref _mesh);
});
```

---

## Non-Renderer Resources

Do **not** dispose unrelated scene resources inside the callback.

Examples include:

- Input
- Gameplay state
- UI state
- Physics objects
- File handles
- Networking
- General managed objects

These should be disposed before or after `Unload()`, depending on the needs of the scene.

---

## Preserving Resources Between Scenes

If a renderer resource should survive the scene transition (for example, a `RenderTexture` passed to the next scene), simply do not dispose it inside the callback.

```csharp
_renderer.Unload(() =>
{
    // Intentionally preserved.
    // _historyTexture.Dispose();

    _framebuffers.Dispose();
    _material.Dispose();
});
```

Ownership is then transferred to the next scene.

---

## Benefits

Using `VulkanRenderer.Unload()` guarantees a consistent scene shutdown sequence and prevents:

- Destroying GPU resources while they are still in use.
- Cached descriptor sets referencing disposed resources.
- Stale material buffer caches surviving scene transitions.
- Resource lifetime bugs caused by incorrect cleanup ordering.

All renderer-owned scene resources should be released through this API.