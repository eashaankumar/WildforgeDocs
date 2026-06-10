# Render Pass Usage

Render passes are created after `BeginFrame()` and executed before `EndFrame()`.

Basic frame flow:

```csharp
renderer.BeginFrame();

var pass = renderer.CreateRasterPass(camera, colorTarget, depthTarget);

pass.Clear(new Vector4(0, 0, 0, 1));
pass.ClearDepth();

pass.DrawMesh(mesh, material, transform);

renderer.Execute(pass);

renderer.EndFrame();
```

---

# Raster Pass

Use a raster pass to render meshes into a color/depth target.

```csharp
var pass = renderer.CreateRasterPass(
    camera,
    colorTexture,
    depthTexture);

pass.Clear(new Vector4(0, 0, 0, 1));
pass.ClearDepth();

pass.DrawMesh(mesh, material, transform);

renderer.Execute(pass);
```

---

# DrawMesh

```csharp
pass.DrawMesh(
    meshHandle,
    material,
    transform);
```

Example:

```csharp
pass.DrawMesh(
    cubeMesh,
    stoneMaterial,
    Matrix4x4.CreateTranslation(0, 0, 0));
```

---

# DrawMeshIndirect

```csharp
pass.DrawMeshIndirect(
    meshHandle,
    material,
    indirectArgsBuffer,
    transform);
```

The indirect buffer must contain a valid:

```csharp
DrawIndexedIndirectCommand
```

---

# Multiple Draws

```csharp
foreach (var entity in entities)
{
    pass.DrawMesh(
        entity.Mesh,
        entity.Material,
        entity.Transform);
}
```

---

# Multiple Passes

```csharp
renderer.BeginFrame();

var geometryPass =
    renderer.CreateRasterPass(
        camera,
        geometryColor,
        geometryDepth);

geometryPass.ClearDepth();

foreach (var entity in visibleEntities)
{
    geometryPass.DrawMesh(
        entity.Mesh,
        entity.Material,
        entity.Transform);
}

var uiPass =
    renderer.CreateRasterPass(
        uiCamera,
        geometryColor,
        geometryDepth);

uiPass.DrawMesh(
    uiMesh,
    uiMaterial,
    Matrix4x4.Identity);

renderer.Execute(
    geometryPass,
    uiPass);

renderer.EndFrame();
```

Passes execute in submission order.

---

# Pass Lifetime

Passes are frame-local objects.

Valid:

```csharp
renderer.BeginFrame();

var pass =
    renderer.CreateRasterPass(
        camera,
        color,
        depth);

pass.DrawMesh(mesh, material, transform);

renderer.Execute(pass);

renderer.EndFrame();
```

Invalid:

```csharp
RasterPass savedPass;

renderer.BeginFrame();

renderer.Execute(savedPass);

renderer.EndFrame();
```

Never store a pass and reuse it in a later frame.

---

# Material Ownership

Materials are no longer globally registered.

Each prepared raster pass owns:

```csharp
RasterMaterialStore Materials
Dictionary<RasterMaterial, int> LocalMaterialIndices
VulkanRasterMaterialBufferManager MaterialBuffers
VulkanBuffer GlobalBuffer
VulkanBuffer InstanceBuffer
List<RasterDrawBatch> DrawBatches
Descriptor Cache
```

Material indices are local to the pass.

Do not assume a material has the same index across passes.

---

# Material Mutation Rule

Allowed:

```csharp
material.SetFloat("Roughness", 0.5f);

pass.DrawMesh(
    mesh,
    material,
    transform);

renderer.Execute(pass);
```

Avoid:

```csharp
pass.DrawMesh(
    mesh,
    material,
    transform);

material.SetFloat("Roughness", 1.0f);
```

After a material has been submitted to a pass, treat it as immutable until execution is complete.

---

# What The Renderer Owns

The renderer owns shared GPU infrastructure:

```csharp
VulkanContext
VulkanDevice
Swapchain
Mesh Store
Texture Store
Pipeline Cache
Framebuffer Cache
```

The renderer does NOT own raster material state anymore.

---

# What The Pass Owns

The pass owns all draw preparation data:

```csharp
Materials
LocalMaterialIndices
MaterialBuffers
GlobalBuffer
InstanceBuffer
DrawBatches
Descriptors
```

This allows passes to be prepared independently and later supports parallel recording.

---

# Clear Color

```csharp
pass.Clear(
    new Vector4(
        0.0f,
        0.0f,
        0.0f,
        1.0f));
```

---

# Clear Depth

```csharp
pass.ClearDepth();
```

---

# Typical Frame

```csharp
renderer.BeginFrame();

var pass =
    renderer.CreateRasterPass(
        camera,
        renderer.FinalFrame,
        depthTexture);

pass.Clear(
    new Vector4(
        0,
        0,
        0,
        1));

pass.ClearDepth();

foreach (var entity in visibleEntities)
{
    pass.DrawMesh(
        entity.Mesh,
        entity.Material,
        entity.Transform);
}

renderer.Execute(pass);

renderer.EndFrame();
```

---

# Internal Raster Pass Pipeline

Execution flow:

```text
DrawMesh()
    ↓
Queued Draw Commands
    ↓
PreparePass()
    ↓
Register Materials Into Pass Material Store
    ↓
Build Packed Material Buffers
    ↓
Patch Instance Material Indices
    ↓
Create/Refresh Descriptor Sets
    ↓
Build Draw Batches
    ↓
Record Vulkan Commands
    ↓
Submit
```

Material indices are assigned during:

```csharp
PreparePass()
```

not during:

```csharp
DrawMesh()
```

## Threading Model

Render passes support **pass-level multithreading**, not **per-call multithreading**.

This means you may build different passes on different threads:

```csharp
var passA = renderer.CreateRasterPass(cameraA, colorA, depthA);
var passB = renderer.CreateRasterPass(cameraB, colorB, depthB);

var taskA = Task.Run(() =>
{
    passA.DrawMesh(meshA, materialA, transformA);
    passA.DrawMesh(meshB, materialB, transformB);
});

var taskB = Task.Run(() =>
{
    passB.DrawMesh(meshC, materialC, transformC);
    passB.DrawMesh(meshD, materialD, transformD);
});

Task.WaitAll(taskA, taskB);

renderer.Execute(passA, passB);