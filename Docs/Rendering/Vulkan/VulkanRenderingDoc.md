# VulkanRenderer Architecture Documentation

# Overview

`VulkanRenderer` is the low-level GPU execution backend for the Forgewild Engine.

It is responsible for:

* Vulkan device/swapchain management
* GPU resource ownership
* Mesh GPU upload + BLAS management
* TLAS construction
* Ray tracing dispatch
* Frame synchronization
* Presentation to the swapchain

It is NOT responsible for:

* Scene ownership
* Entities/components
* Gameplay state
* World simulation
* Determining what exists in the world

The renderer is a pure GPU backend/service layer.

---

# Core Architecture Philosophy

## Ownership Model

### Scene owns:

* entities
* transforms
* gameplay state
* logical world contents
* object lifetime
* world truth

### Renderer owns:

* GPU resources
* MeshStore
* VulkanMeshStore
* BLAS/TLAS backend resources
* pipelines/descriptors
* synchronization
* rendering execution
* GPU representation of the world

---

# Scene → Renderer Relationship

Scene submits intent into renderer.

Renderer never queries scene.

Renderer never owns gameplay state.

This separation is critical.

Correct architecture:

```text
Scene decides WHAT exists.

Renderer decides HOW it is rendered.
```

---

# High-Level Rendering Flow

Main loop:

```csharp
scene.Update();

renderer.Render();
```

The renderer no longer receives temporary per-frame RT draw commands.

Instead, the scene creates persistent RT instances and updates them only when necessary.

---

# Retained RT Instance Architecture

## Creating RT Instances

Scene creates RT instances once:

```csharp
_triangleRtHandle =
    renderer.AddRtInstance(mesh, transform);
```

Renderer returns a stable handle:

```csharp
RtInstanceHandle
```

This handle represents persistent RT instance identity.

---

## Updating RT Transforms

When an object moves:

```csharp
renderer.SetRtInstanceTransform(
    _triangleRtHandle,
    transform);
```

This updates the renderer’s GPU-side RT scene representation.

No scene scanning is required.

---

## Removing RT Instances

When an object is destroyed:

```csharp
renderer.RemoveRtInstance(_triangleRtHandle);
```

Renderer removes the RT instance from future TLAS rebuilds.

---

# Why Retained RT Instances Exist

The old immediate-mode approach:

```csharp
renderer.TraceMesh(mesh, transform);
```

had major problems:

* rebuilt temporary RT lists every frame
* no persistent identity
* expensive transform comparison
* renderer had to rediscover scene state constantly
* difficult TLAS optimization path

The retained instance model fixes this.

---

# Retained RT Instance Storage

Renderer internally stores:

```csharp
private struct RtInstance
{
    public MeshHandle Mesh;
    public Matrix4x4 Transform;
    public bool Alive;
    public int Generation;
}
```

This allows:

* stable RT instance identity
* efficient transform updates
* future TLAS refit/update support
* O(1) transform modification
* no full-scene transform scans

---

# Mesh Architecture

## MeshStore

CPU-side mesh repository.

Renderer-owned.

Stores:

* mesh layouts
* vertex/index CPU data
* mesh handles

Does NOT contain Vulkan objects.

---

## VulkanMeshStore

GPU-side mesh backend.

Renderer-owned.

Responsible for:

* vertex/index GPU buffers
* BLAS creation
* device addresses
* GPU upload synchronization

Meshes are lazily uploaded when first used.

---

# BLAS vs TLAS

## BLAS (Bottom Level Acceleration Structure)

Represents geometry of a single mesh.

One BLAS per mesh.

Owned internally by `VulkanMeshStore`.

Users/scenes never directly manage BLAS.

---

## TLAS (Top Level Acceleration Structure)

Represents the persistent RT scene instance list.

Contains:

* references to BLAS
* instance transforms
* instance metadata

Owned internally by `VulkanRenderer`.

---

# RT Instance Handles

Renderer exposes:

```csharp
RtInstanceHandle
```

Internally:

```csharp
public readonly struct RtInstanceHandle
{
    public readonly int Index;
    public readonly int Generation;
}
```

Purpose:

* stable instance identity
* stale handle protection
* safe reuse of freed instance slots

---

# RT Instance Lifetime

## Add

```text
Scene creates RT object
→ AddRtInstance()
→ renderer allocates persistent RT instance
→ TLAS rebuild required
```

---

## Transform Change

```text
Scene moves object
→ SetRtInstanceTransform()
→ renderer marks RT transform dirty
→ TLAS update/rebuild required
```

---

## Remove

```text
Scene destroys object
→ RemoveRtInstance()
→ renderer frees RT instance slot
→ TLAS rebuild required
```

---

# TLAS Synchronization

Renderer tracks two RT synchronization states:

## Full Rebuild Required

Triggered when:

* RT instance added
* RT instance removed

This changes TLAS membership/layout.

Renderer performs full TLAS rebuild.

---

## Transform Update Required

Triggered when:

* RT instance transform changes

Current implementation:

```text
transform change
→ full TLAS rebuild
```

Future implementation:

```text
transform change
→ TLAS UpdateKhr/refit
```

---

# Current TLAS Rebuild Flow

Renderer:

1. Retires old RT resources
2. Builds instance buffer
3. Builds TLAS
4. Recreates RT descriptors/pipeline

---

# Future TLAS Optimization

Current:

* RT instance add/remove
  → full rebuild

* RT transform change
  → full rebuild

Future:

* RT instance add/remove
  → full rebuild

* RT transform change
  → TLAS UpdateKhr/refit

This will allow moving objects to update efficiently without rebuilding the entire TLAS.

---

# Deferred GPU Destruction

## Problem

GPU execution is asynchronous.

If CPU destroys:

* TLAS
* descriptor sets
* RT pipeline
* scratch buffers
* instance buffers

while GPU commands still reference them:

```text
GPU use-after-free
→ DeviceLost
```

This caused the original renderer crash.

---

# Deferred Destruction Solution

Renderer uses per-frame deferred destruction queues:

```csharp
_deferredDestroys[frameIndex]
```

Old RT resources are retired instead of immediately destroyed.

Resources are destroyed only after that frame’s fence signals.

This guarantees GPU safety.

---

# Deferred RT Resources

Renderer defers destruction of:

* TLAS
* TLAS buffers
* scratch buffers
* VulkanDescriptors
* VulkanRayTracingPipeline
* instance buffers

This ensures descriptors/pipelines never outlive the TLAS they reference.

---

# Frame Synchronization

Each frame owns:

```text
Fence
ImageAvailable semaphore
RenderFinished semaphore
CommandBuffer
```

---

# Frame Flow

Per frame:

```text
Wait frame fence
Flush deferred destroys
Sync TLAS
Acquire swapchain image
Reset fence
Record commands
Submit
Present
Advance frame index
```

---

# EnsureRtObjects()

RT descriptors/pipeline are lazily created.

Reason:

* TLAS must already exist

Objects:

* VulkanDescriptors
* VulkanRayTracingPipeline

They are recreated whenever TLAS changes.

---

# Ray Tracing Dispatch

Inside `RecordTraceAndCopy()`:

1. Bind RT pipeline
2. Bind descriptor sets
3. Dispatch rays
4. Copy output image → swapchain image
5. Restore image layouts

---

# Output Image Strategy

Renderer ray traces into:

```text
_outputImage
```

Then copies into swapchain image.

Advantages:

* decouples RT output from swapchain
* future post-processing support
* accumulation/history buffer support
* hybrid raster/RT compositing support

---

# Swapchain Flow

Per frame:

```text
Acquire swapchain image
Trace rays into output image
Copy output image → swapchain image
Present
```

---

# Why Scene Does Not Own GPU Resources

Incorrect architecture:

```text
Scene owns TLAS
Scene owns Vulkan buffers
Scene owns BLAS
Scene manages synchronization
```

Problems:

* gameplay coupled to GPU backend
* backend abstraction impossible
* synchronization leaks into gameplay
* GPU lifetime bugs everywhere
* resource ownership chaos

Correct architecture:

```text
Scene owns gameplay truth.

Renderer owns GPU representation.
```

---

# Current VulkanRenderer Responsibilities

## VulkanRenderer owns:

* VulkanContext
* VulkanSwapchain
* VulkanFrames
* MeshStore
* VulkanMeshStore
* BLAS cache
* TLAS
* retained RT instance state
* RT descriptors
* RT pipeline
* output image
* synchronization
* deferred destruction queues

---

# VulkanRenderer does NOT own:

* scenes
* entities
* gameplay logic
* authoritative transforms
* world simulation

---

# Planned Raster Path

Eventually:

```csharp
renderer.DrawMesh(mesh, material, transform);
```

will coexist with:

```csharp
renderer.AddRtInstance(...)
renderer.SetRtInstanceTransform(...)
```

Both systems will share:

* MeshStore
* VulkanMeshStore
* mesh GPU resources

---

# Planned Deferred Resource System

Current:

* manual deferred RT destruction queues

Future:

* generalized renderer-wide deferred destruction system

Used for:

* buffers
* images
* pipelines
* descriptor pools
* acceleration structures
* transient GPU allocations

---

# Planned Post Processing

Current:

```text
RT output
→ swapchain
```

Future:

```text
RT output
→ post FX
→ compositing
→ UI
→ present
```

---

# Architectural Summary

## Scene Layer

High-level gameplay/world layer.

Owns:

* entities
* transforms
* gameplay state
* world logic

Submits:

* RT instance creation/removal
* RT transform updates
* future raster draw commands

---

## Renderer Layer

Low-level GPU execution layer.

Owns:

* Vulkan resources
* synchronization
* pipelines
* acceleration structures
* GPU scene representation

Executes:

* rasterization
* ray tracing
* presentation

---

# Final Philosophy

```text
Scene owns world truth.

Renderer owns GPU representation of that truth.
```

Scene explicitly notifies renderer when RT state changes.

Renderer never discovers gameplay changes by scanning scene state.
