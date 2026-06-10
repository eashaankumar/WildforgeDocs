# Raster Instance API

Raster should mirror the current RT scene-facing API:

```csharp
var material = new RasterMaterial(renderer.DefaultRasterPipeline);
material.SetVector3("BaseColor", new Vector3(1, 0, 0));

var handle = renderer.AddRasterInstance(mesh, material, transform);
renderer.SetRasterInstanceTransform(handle, newTransform);
```

The raster public model is:

- `RasterShader`: one independent stage, vertex or fragment.
- `RasterPipeline`: vertex + fragment + fixed-function `RasterPipelineState`.
- `RasterMaterial`: reflected property packer for the pipeline material layout.
- `RasterInstanceHandle`: mirrors `RtInstanceHandle`.

The `.rasterpipeline` asset is kept for fixed-function hints. Reflection still comes from shader SPIR-V/reflection JSON.
