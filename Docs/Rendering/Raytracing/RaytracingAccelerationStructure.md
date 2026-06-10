# Raytracing Acceleration Structures (RTAS)

Raytracing instances are stored inside a persistent `RaytracingAccelerationStructure` (RTAS).

Unlike raster rendering, raytracing geometry is not submitted every frame. The RTAS owns all raytracing instances and manages BLAS/TLAS lifetime internally.

A `RaytracingPass` is a temporary render command object that references an RTAS.

---

# Creating an RTAS

Create an RTAS once and keep it alive for the lifetime of the scene.

```csharp
private RaytracingAccelerationStructure _rtas;

public override void Load()
{
    _rtas = _renderer.CreateRaytracingAccelerationStructure();
}
```

Destroy it when the scene unloads.

```csharp
public override void Unload()
{
    _rtas.Dispose();
}
```

---

# Adding Instances

Instances are added once and return a persistent handle.

```csharp
RtInstanceHandle redHandle = _rtas.AddInstance(
    mesh,
    redMaterial,
    redTransform);
```

```csharp
RtInstanceHandle blueHandle = _rtas.AddInstance(
    mesh,
    blueMaterial,
    blueTransform);
```

The returned handle remains valid until the instance is removed.

---

# Updating Transforms

Update transforms without re-adding instances.

```csharp
_rtas.SetInstanceTransform(
    redHandle,
    redTransform);
```

```csharp
_rtas.SetInstanceTransform(
    blueHandle,
    blueTransform);
```

Transform changes automatically update the TLAS.

---

# Updating Materials

```csharp
_rtas.SetInstanceMaterial(
    redHandle,
    newMaterial);
```

---

# Removing Instances

```csharp
_rtas.RemoveInstance(redHandle);
```

After removal the handle becomes invalid.

```csharp
if (handle.IsValid)
{
    ...
}
```

should not be used after removal.

---

# Creating a Raytracing Pass

A raytracing pass references an RTAS.

```csharp
var rtPass = _renderer.CreateRaytracingPass(
    _camera,
    _rtas,
    _rtColor,
    _rtDepth);
```

---

# Clearing

Clear the raytracing output texture.

```csharp
rtPass.Clear(Color.Black);
```

If RT depth is implemented as an `R32_Float` storage texture, do not use depth attachment clears.

Instead use the dedicated RT depth clear API if available.

```csharp
rtPass.ClearDepth(float.PositiveInfinity);
```

---

# Executing

Execute a raytracing pass by itself:

```csharp
_renderer.Execute(rtPass);
```

Or alongside raster passes:

```csharp
_renderer.Execute(
    worldPass,
    rtPass,
    uiPass);
```

---

# Complete Example

## Initialization

```csharp
private RaytracingAccelerationStructure _rtas;

private RtInstanceHandle _redHandle;
private RtInstanceHandle _blueHandle;

public override void Load()
{
    _rtas = _renderer.CreateRaytracingAccelerationStructure();

    _redHandle = _rtas.AddInstance(
        _triangleMesh,
        _redRtMaterial,
        Matrix4x4.Identity);

    _blueHandle = _rtas.AddInstance(
        _triangleMesh,
        _blueRtMaterial,
        Matrix4x4.Identity);
}
```

## Per Frame

```csharp
_rtas.SetInstanceTransform(
    _redHandle,
    redTransform.ToMatrix());

_rtas.SetInstanceTransform(
    _blueHandle,
    blueTransform.ToMatrix());

var rtPass = _renderer.CreateRaytracingPass(
    _camera,
    _rtas,
    _rtColor,
    _rtDepth);

rtPass.Clear(Color.Black);

_renderer.Execute(rtPass);
```

## Cleanup

```csharp
public override void Unload()
{
    _rtas.Dispose();
}
```

---

# RTAS vs Raytracing Pass

RTAS owns scene data:

```text
Meshes
Materials
Instance Transforms
BLAS
TLAS
```

RaytracingPass owns render commands:

```text
Camera
Output Color
Output Depth
Clear Operations
Dispatch
```

Typical lifetime:

```text
Scene Start
 └─ Create RTAS

Scene Runtime
 ├─ AddInstance()
 ├─ RemoveInstance()
 ├─ SetInstanceTransform()
 └─ CreateRaytracingPass() every frame

Scene End
 └─ Dispose RTAS
```