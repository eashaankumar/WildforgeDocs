# ForgeCompute Shader Documentation

ForgeCompute files use the `.forgecompute` extension. They use the same core shader language as ForgeShader, but they
are compiled as compute shaders instead of raster vertex/fragment shaders.

## Basic file structure

```hlsl
Name GenerateTriangleInstances
Kernel CSMain

struct TriangleInstance
{
    float4x4 LocalToWorld;
    float4 Color;
};

Buffer<TriangleInstance> Instances;
float4 _startColor;
float4 _endColor;

[numthreads(64, 1, 1)]
void CSMain(uint3 id : ThreadId)
{
    uint index = id.x;

    float t = 0.0;

    if (_dispatchThreadCount.x > 1)
        t = (float)index / (float)(_dispatchThreadCount.x - 1);

    float3 position = float3(
        ((float)index - (float)_dispatchThreadCount.x * 0.5) * 1.25,
        0.0,
        10.0);

    TriangleInstance instance;

    instance.LocalToWorld = float4x4(
        1, 0, 0, position.x,
        0, 1, 0, position.y,
        0, 0, 1, position.z,
        0, 0, 0, 1
    );

    instance.Color = lerp(_startColor, _endColor, t);

    Instances[index] = instance;
}
```

## Required declarations

Every `.forgecompute` file needs:

```hlsl
Name ShaderName
Kernel KernelFunctionName
```

`Name` names the compiled compute asset. `Kernel` selects the function that becomes the compute entry point.

The kernel must use this shape:

```hlsl
[numthreads(x, y, z)]
void KernelName(uint3 id : ThreadId)
{
}
```

`ThreadId` maps to the global dispatch thread id.

## Thread count

`[numthreads(x, y, z)]` defines the number of threads per workgroup.

Example:

```hlsl
[numthreads(64, 1, 1)]
```

A dispatch call of:

```csharp
compute.Dispatch(1, 1, 1);
```

runs 64 total threads.

A dispatch call of:

```csharp
compute.Dispatch(4, 1, 1);
```

runs 256 total threads.

## Built-in compute variables

ForgeCompute supports these built-ins:

```hlsl
_dispatchThreadCount
_dispatchThreadId
_groupId
_groupThreadId
_threadId
```

They map to native GLSL/Vulkan compute values.

| ForgeCompute           | Meaning                                                                                         |
|------------------------|-------------------------------------------------------------------------------------------------|
| `_dispatchThreadCount` | Total number of dispatched threads. Equivalent to workgroup count multiplied by workgroup size. |
| `_dispatchThreadId`    | Global dispatch thread id.                                                                      |
| `_threadId`            | Alias for global dispatch thread id.                                                            |
| `_groupId`             | Current workgroup id.                                                                           |
| `_groupThreadId`       | Thread id inside the current workgroup.                                                         |

Example:

```hlsl
uint index = _dispatchThreadId.x;
```

or:

```hlsl
void CSMain(uint3 id : ThreadId)
{
    uint index = id.x;
}
```

## Buffers

Buffers are declared openly at the top level:

```hlsl
Buffer<TriangleInstance> Instances;
```

A `Buffer<T>` becomes a storage buffer. It can be read from and written to by the compute shader.

Example:

```hlsl
Instances[index] = instance;
```

Runtime usage:

```csharp
compute.SetBuffer("Instances", instanceBuffer);
compute.Dispatch(groupsX, groupsY, groupsZ);
```

## Loose top-level parameters

Loose top-level values are compute dispatch parameters:

```hlsl
float4 _startColor;
float4 _endColor;
```

They are not raster `Properties`. They are not local variables. They are per-dispatch parameters supplied from C#.

Runtime usage:

```csharp
compute.SetVector4("_startColor", new Vector4(1, 0, 0, 1));
compute.SetVector4("_endColor", new Vector4(0, 0, 1, 1));
```

## Local variables

Local variables still exist inside functions:

```hlsl
void CSMain(uint3 id : ThreadId)
{
    float t = 0.0;
    uint index = id.x;
}
```

Top-level loose variables are dispatch parameters. Function-scope variables are normal locals.

## Structs

Structs are supported:

```hlsl
struct TriangleInstance
{
    float4x4 LocalToWorld;
    float4 Color;
};
```

Structs can be used inside buffers:

```hlsl
Buffer<TriangleInstance> Instances;
```

## Helper functions

Helper functions are supported as long as they are not the selected `Kernel` function.

Example:

```hlsl
float Remap01(float value, float minValue, float maxValue)
{
    return saturate((value - minValue) / (maxValue - minValue));
}
```

## Common functions

ForgeCompute uses the same core math language as ForgeShader. Common functions include:

```hlsl
abs
clamp
min
max
lerp
sin
cos
dot
cross
normalize
length
sqrt
pow
smoothstep
```

`lerp(a, b, t)` is translated to the backend equivalent.

## Compute to indirect draw workflow

A common pattern is:

```text
Compute shader writes instance buffer
Compute shader writes indirect args buffer
Renderer draws with DrawMeshIndirect
```

Example C# shape:

```csharp
var compute = new ComputeShader(
    renderer,
    Path.Combine(EnginePaths.ShaderRoot, "Compute", "GenerateTriangleInstances.forgecompute.bin"));

compute.SetBuffer("Instances", instanceBuffer);
compute.SetVector4("_startColor", new Vector4(1, 0, 0, 1));
compute.SetVector4("_endColor", new Vector4(0, 0, 1, 1));
compute.Dispatch(1, 1, 1);

renderer.DrawMeshIndirect(mesh, material, argsBuffer, transform);
```

The renderer is responsible for transitioning buffers from compute-write usage to shader-read or indirect-argument usage
before raster rendering.

## Notes and limitations

`Properties { }` is not used in `.forgecompute` files.

Loose top-level parameters are dispatch parameters.

Special engine globals may later use an explicit marker such as `@global`, but that is separate from normal compute
dispatch parameters.

Bounds checks are the shader author's responsibility. If the dispatch runs more threads than the buffer contains, the
shader can write out of bounds. Use dispatch counts and buffer sizes carefully.

