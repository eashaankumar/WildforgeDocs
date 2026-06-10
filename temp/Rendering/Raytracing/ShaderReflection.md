# Ray Tracing Shader Reflection

This is the first reflection layer for the C# renderer.

## Current scope

The engine now has renderer-independent metadata classes:

- `ShaderReflection`
- `ShaderResourceBinding`
- `ShaderStructLayout`
- `ShaderField`
- `ShaderStage`
- `ShaderDescriptorType`

`RTShader` owns a `ShaderReflection` object and each `RTShaderHitGroup` may optionally own a `MaterialLayout`.

## Reflection source

For now, reflection is loaded from offline JSON:

```csharp
RTShader shader = RTShader.LoadWithReflectionJson(
    name: "DefaultTriangleRT",
    rayGenPath: "Shaders/rt_triangle.rgen.spv",
    missPath: "Shaders/rt_triangle.rmiss.spv",
    hitGroups:
    [
        new RTShaderHitGroup(
            name: "DefaultLit",
            closestHitPath: "Shaders/rt_triangle.rchit.spv")
    ],
    reflectionJsonPath: "Shaders/rt_triangle.reflection.json");
```

A later shader compilation step should run SPIRV-Reflect or SPIRV-Cross and emit this JSON automatically.

## Why JSON first

This keeps engine startup simple and lets reflection be tested without binding the runtime to a native reflection library. The runtime consumes reflection metadata; the build pipeline produces it.

## Renderer connection

`VulkanRayTracingPipeline` now receives an `RTShader` instead of hardcoding shader paths. It builds stages and hit groups from:

```csharp
RTShader.RayGenPath
RTShader.MissPath
RTShader.HitGroups
```

The SBT order is:

```text
0: raygen
1: miss
2 + hitGroupIndex: hit group
```

TLAS `InstanceShaderBindingTableRecordOffset` should use the hit-group index relative to the first hit group, not the absolute SBT group index.

## Vulkan descriptor layout bridge

`VulkanDescriptors` now consumes `RTShader.Reflection` and builds descriptor set layouts from reflected bindings instead of hardcoding the RT layout in the Vulkan backend.

Current behavior:

- Reflected bindings are grouped by descriptor set.
- Sets must be dense (`0..N-1`) for this first implementation.
- Each `ShaderResourceBinding` maps to a `DescriptorSetLayoutBinding`.
- `VulkanRayTracingPipeline` receives all reflected descriptor set layouts and uses them to create the pipeline layout.
- `RecordTraceAndCopy` binds all descriptor sets produced from reflection.

Still intentionally manual:

- Actual descriptor writes only handle the current known RT resources:
  - `topLevelAS` / `tlas` / `sceneTlas`
  - `outputImage` / `output` / `raytraceOutput`
- Storage buffers, sampled images, samplers, and material buffers are reflected into layouts but are not generically bound yet.

Next step: add a generic reflected resource binding API, then use reflected struct layouts for `RTMaterial` packing.
