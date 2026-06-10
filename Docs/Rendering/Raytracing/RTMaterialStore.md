# RTMaterialStore

`RTMaterialStore` is the CPU-side bridge between `RTMaterial` and future Vulkan material buffers.

It does three things:

1. Assigns each `RTMaterial` a stable global material index.
2. Groups materials by `RTShader + HitGroupIndex + ShaderStructLayout`.
3. Builds dense packed CPU byte buffers for each reflected material layout.

This preserves the Forgewild RT material rule:

```text
Do not use one universal material struct.
Use one material buffer/layout per hit group when the hit group needs one.
```

Example future flow:

```csharp
RTMaterial glass = new(renderer.DefaultRTShader, "Glass");
glass.SetFloat("IOR", 1.5f);
glass.SetFloat("Roughness", 0.02f);

int globalIndex = renderer.RTMaterials.Register(glass);
renderer.RTMaterials.MarkDirty(glass);

IReadOnlyList<RTMaterialPackedBuffer> buffers = renderer.RTMaterials.RebuildPackedBuffers();
RTMaterialPackedLocation location = renderer.RTMaterials.GetPackedLocation(globalIndex);
```

`RTMaterialPackedBuffer.Data` is ready for the Vulkan layer to upload into a storage buffer.

`RTMaterialPackedLocation.LocalMaterialIndex` is the index the shader should use when reading from that packed buffer.

This step does **not** create GPU buffers yet. The next renderer milestone is:

```text
RTMaterialStore packed buffers
    -> Vulkan storage buffers
    -> reflected descriptor binding
    -> instance data uses local material index
```
