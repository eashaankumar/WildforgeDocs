# ForgeShader / ForgeCompute Casting Rules

ForgeShader and ForgeCompute use **GLSL-style constructor casts**.

Do **not** use C-style or HLSL-style casts.

## Correct casting syntax

Use the target type as a function-like constructor:

```hlsl
float(index)
uint(value)
int(value)
bool(value)
float3(position)
float4(color, 1.0)
float4x4(...)
```

This matches GLSL syntax and is the expected Forge shader syntax.

## Incorrect casting syntax

Do not write:

```hlsl
(float)index
(uint)value
(int)value
(bool)value
(float3)position
```

These are C/HLSL-style casts and will not compile correctly through the Forge GLSL backend.

## Examples

### Integer to float

Correct:

```hlsl
float t = float(index) / float(count - 1);
```

Incorrect:

```hlsl
float t = (float)index / (float)(count - 1);
```

### Unsigned integer conversion

Correct:

```hlsl
uint instanceIndex = uint(id.x);
```

Incorrect:

```hlsl
uint instanceIndex = (uint)id.x;
```

### Vector construction

Correct:

```hlsl
float3 position = float3(x, y, z);
float4 color = float4(rgb, 1.0);
```

Incorrect:

```hlsl
float3 position = (float3)value;
```

## Compute shader example

Correct ForgeCompute style:

```hlsl
[numthreads(64, 1, 1)]
void CSMain(uint3 id : ThreadId)
{
    uint index = id.x;

    float t = 0.0;

    if (_dispatchThreadCount.x > 1)
        t = float(index) / float(_dispatchThreadCount.x - 1);

    float3 position = float3(
        (float(index) - float(_dispatchThreadCount.x) * 0.5) * 1.25,
        0.0,
        10.0);
}
```

## Why Forge uses this rule

Forge shaders are transpiled to GLSL for Vulkan. GLSL does not use C-style casts. It uses constructor-style conversion
instead:

```glsl
float(x)
uint(x)
vec3(x, y, z)
mat4(...)
```

Keeping Forge syntax aligned with GLSL avoids ambiguous cast rewriting and prevents backend-specific bugs.

## Rule of thumb

When converting a value, always write:

```hlsl
TargetType(value)
```

Never write:

```hlsl
(TargetType)value
```
