# RT material GPU buffer upload

This step connects the CPU-side `RTMaterialStore` to Vulkan storage buffers.

Current flow:

```text
RTMaterial
  -> RTMaterialStore.RebuildPackedBuffers()
  -> RTMaterialPackedBuffer[]
  -> VulkanRTMaterialBufferManager
  -> Vulkan storage buffers
  -> VulkanDescriptors material storage-buffer writes
```

## What was added

`VulkanRTMaterialBufferManager` owns one Vulkan storage buffer per packed material batch.

Each batch still follows the engine rule:

```text
shader + hit group + material layout -> one material buffer
```

That means `GlassMaterial`, `FoliageMaterial`, and `DefaultLitMaterial` do not need to share one giant universal struct.

## Descriptor matching

Until reflection contains explicit `binding -> material layout` metadata, descriptor matching is name-based:

```text
GlassMaterial
GlassMaterials
GlassMaterialBuffer
GlassMaterialsBuffer
```

all match a packed material layout named `GlassMaterial`.

There is also a temporary fallback: if exactly one material buffer exists and the reflected binding name contains `material`, that buffer is used.

## What this does not solve yet

This does not yet implement a final generic descriptor-binding system. It only bridges reflected storage-buffer bindings to material buffers when the binding name is recognizable.

Next step:

```text
reflection JSON metadata:
  materialBuffers:
    binding name -> hit group/material layout
```

After that, descriptor writes should be fully metadata-driven instead of name-based.
