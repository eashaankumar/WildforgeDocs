# ForgeShader

ForgeShader is the custom shader language used by the Forgewild Engine.

ForgeShader combines ideas from GLSL, HLSL, and Unity ShaderLab while providing a simpler, engine-focused authoring
experience.

ForgeShader source files use the `.forgeshader` extension and compile into platform-specific shaders (currently Vulkan
GLSL/SPIR-V).

# Usage

```c
 .\Tools\Forgewild.ForgeShader.exe *.forgeshader   
```

---

# Example

```c
Name DefaultTriangleRaster
Vertex vert
Fragment frag

PrimitiveTopology TriangleList
CullMode Back
FrontFace CounterClockwise
PolygonMode Fill
BlendMode Opaque
DepthTest True
DepthWrite True
DepthCompareOp LessOrEqual

struct VertIn
{
    float3 pos : POSITION;
};

struct FragIn
{
};

struct FragOut
{
    float4 color;
};

Properties
{
    float3 BaseColor;
    float Roughness;
} // No semicolor


FragIn vert(VertIn vert)
{
    float4 world = _objectToWorld * float4(vert.pos, 1.0);
    _position = _viewProjection * world;

    FragIn frag;
    return frag;
}

FragOut frag(FragIn fragIn)
{    
    FragOut fragOut;
    fragOut.color = float4(Properties.BaseColor, 1.0f);
    
    return fragOut;
}
```

---

# Reserved Keywords

```c
// Shader Definition
Name
Vertex
Fragment
Compute

// Semantics
POSITION
NORMAL

TANGENT
BITANGENT

TEXCOORD0
TEXCOORD1
TEXCOORD2
TEXCOORD3

COLOR0
COLOR1

BLENDINDICES
BLENDWEIGHT

// Properties
Properties

// Shader Modes
PrimitiveTopology
CullMode
FrontFace
PolygonMode
BlendMode

DepthTest
DepthWrite
DepthCompareOp

True
False

// Syntax
struct

// Resource Ownership
local
global

// Renderer Resources
color
depth
normal
shadow
```

---

# Shader Declaration

```c
Name <ShaderName>
Vertex <VertexEntryPoint>
Fragment <FragmentEntryPoint>
```

Example:

```c
Name DefaultTriangleRaster
Vertex vert
Fragment frag
```

---

# Pipeline State

## PrimitiveTopology

```c
PrimitiveTopology TriangleList
```

Valid values:

```text
TriangleList
LineList
LineStrip
PointList
```

---

## CullMode

```c
CullMode Back
```

Valid values:

```text
None
Front
Back
FrontAndBack
```

---

## FrontFace

```c
FrontFace CounterClockwise
```

Valid values:

```text
CounterClockwise
Clockwise
```

---

## PolygonMode

```c
PolygonMode Fill
```

Valid values:

```text
Fill
Line
Point
```

---

## BlendMode

```c
BlendMode Opaque
```

Valid values:

```text
Opaque
AlphaBlend
PremultipliedAlpha
Additive
Multiply
Screen
AlphaClip
```

---

## DepthTest

```c
DepthTest True
```

Valid values:

```text
True
False
```

---

## DepthWrite

```c
DepthWrite True
```

Valid values:

```text
True
False
```

---

## DepthCompareOp

```c
DepthCompareOp LessOrEqual
```

Valid values:

```text
Never
Less
Equal
LessOrEqual
Greater
NotEqual
GreaterOrEqual
Always
```

---

# Built-In Globals

## Properties

Represents the current material instance.

Fields are generated from the shader's Properties block.

Example:

```c
Properties.BaseColor
Properties.Roughness
```

## Instance

```c
uint _instanceID;
uint _vertexID;
```

---

## Object Space

```c
float4x4 _objectToWorld;
float4x4 _worldToObject;
```

---

## Camera

```c
float4x4 _view;
float4x4 _projection;
float4x4 _viewProjection;

float4x4 _inverseView;
float4x4 _inverseProjection;

float3 _cameraPosition;
```

---

## Screen

```c
float2 _resolution;
float2 _invResolution;

float2 _screenUV;
float4 _screenPosition;
```

---

## Time

```c
float _time;
float _deltaTime;

uint _frameIndex;
```

---

## Vertex Outputs

```c
float4 _position;
```

---

## Compute

```c
uint3 _dispatchThreadID;
uint3 _groupThreadID;
uint3 _groupID;
uint3 _threadID;
```

---

## Lighting

```c
float3 _lightDirection;
float3 _lightColor;
```

---

# Resource Types

## Read-Only Buffer

```c
RBuffer<T>
```

Example:

```c
RBuffer<MyData> data;
```

---

## Read-Write Buffer

```c
RWBuffer<T>
```

Example:

```c
RWBuffer<MyData> data;
```

---

## Texture2D

```c
Texture2D
```

Example:

```c
Texture2D Albedo;
```

---

## TextureCube

```c
TextureCube
```

Example:

```c
TextureCube Skybox;
```

---

## Read-Write Texture

```c
RWTexture2D
```

Example:

```c
RWTexture2D Output;
```

---

# Properties

Each raster shader may declare a single Properties block.

```c
Properties
{
    float3 BaseColor;
    float Roughness;
}
```

Properties are automatically backed by the engine's material system.

Properties are accessed through the built-in `Properties` namespace.

```c
float3 color = Properties.BaseColor;
float roughness = Properties.Roughness;
```

ForgeShader automatically resolves the correct material instance for the current draw call.

Users never manually declare material buffers or material indices.

---

# Resource Ownership

ForgeShader resources have explicit ownership.

Ownership determines who is responsible for providing the resource at runtime.

---

## Material Resources

Resources declared inside a Properties block are owned by the material.

```c
Properties
{
    Texture2D Albedo;
    Texture2D Normal;

    float Roughness;
}
```

Access:

```c
Properties.Albedo
Properties.Normal
Properties.Roughness
```

Material resources are supplied through the material API.

```csharp
material.SetTexture("Albedo", albedoTexture);
material.SetTexture("Normal", normalTexture);
material.SetFloat("Roughness", 0.5f);
```

---

## Local Resources

Resources declared outside Properties are local to the shader.

```c
Texture2D Noise;
```

Equivalent:

```c
@local
Texture2D Noise;
```

Local resources are supplied directly by the user.

```csharp
material.SetTexture("Noise", noiseTexture);
```

The `@local` attribute is optional and exists primarily for readability and searching.

---

## Global Resources

Global resources are shared across the renderer.

```c
@global
Texture2D BlueNoise;

@global
TextureCube Skybox;
```

Global resources are supplied through the renderer.

```csharp
renderer.SetGlobalTexture("BlueNoise", blueNoise);
renderer.SetGlobalTexture("Skybox", skybox);
```

Global resources are discovered at shader build time and assigned deterministic bindings.

All shaders referencing the same global resource name share the same renderer-owned value.

---

## Renderer-Injected Resources

Some resources are automatically provided by the renderer.

### Color Buffer

```c
@color
Texture2D SceneColor;
```

Provides access to the current scene color buffer.

---

### Depth Buffer

```c
@depth
Texture2D SceneDepth;
```

Provides access to the current scene depth buffer.

---

### Normal Buffer

```c
@normal
Texture2D SceneNormals;
```

Provides access to the current scene normal buffer.

---

### Shadow Buffer

```c
@shadow
Texture2D MainShadowMap;
```

Provides access to the renderer's shadow map.

Additional shadow helper functions may be provided by the renderer.

---

## Ownership Summary

| Declaration                     | Owner    |
|---------------------------------|----------|
| Properties.Albedo               | Material |
| Texture2D Noise                 | Shader   |
| @local Texture2D Noise          | Shader   |
| @global Texture2D BlueNoise     | Renderer |
| @color Texture2D SceneColor     | Renderer |
| @depth Texture2D SceneDepth     | Renderer |
| @normal Texture2D SceneNormals  | Renderer |
| @shadow Texture2D MainShadowMap | Renderer |

---

# Entry Points

Vertex and fragment shaders are regular functions.

```c
FragIn vert(VertIn input)
{
    ...
}

FragOut frag(FragIn input)
{
    ...
}
```

---

# Standard Types

## Scalar Types

```c
bool

int
uint

float
double
```

---

## Vector Types

```c
bool2
bool3
bool4

int2
int3
int4

uint2
uint3
uint4

float2
float3
float4

double2
double3
double4
```

---

## Matrix Types

```c
float2x2
float3x3
float4x4

double2x2
double3x3
double4x4
```

---

# Vertex Semantics

Vertex input structures may use semantics to bind fields to mesh vertex data.

Example:

```c
struct VertIn
{
    float3 pos      : POSITION;
    float3 normal   : NORMAL;

    float4 tangent  : TANGENT;

    float2 uv       : TEXCOORD0;
    float2 uv1      : TEXCOORD1;

    float4 color    : COLOR0;
};
```

---

## Available Semantics

### Position

```text
POSITION
```

Example:

```c
float3 pos : POSITION;
```

---

### Normal

```text
NORMAL
```

Example:

```c
float3 normal : NORMAL;
```

---

### Tangent

```text
TANGENT
```

Example:

```c
float4 tangent : TANGENT;
```

---

### Bitangent

```text
BITANGENT
```

Example:

```c
float3 bitangent : BITANGENT;
```

---

### Texture Coordinates

```text
TEXCOORD0
TEXCOORD1
TEXCOORD2
TEXCOORD3
TEXCOORD4
TEXCOORD5
TEXCOORD6
TEXCOORD7
```

Examples:

```c
float2 uv  : TEXCOORD0;
float2 uv1 : TEXCOORD1;
float2 uv2 : TEXCOORD2;
float2 uv3 : TEXCOORD3;
```

---

### Vertex Colors

```text
COLOR0
COLOR1
```

Examples:

```c
float4 color  : COLOR0;
float4 color1 : COLOR1;
```

---

### Skinning

```text
BLENDINDICES
BLENDWEIGHT
```

Examples:

```c
uint4 boneIndices : BLENDINDICES;
float4 boneWeights : BLENDWEIGHT;
```

---

## Supported Types

### POSITION

```c
float2 : POSITION
float3 : POSITION
float4 : POSITION
```

---

### NORMAL

```c
float3 : NORMAL
```

---

### TANGENT

```c
float4 : TANGENT
```

---

### BITANGENT

```c
float3 : BITANGENT
```

---

### TEXCOORD0 - TEXCOORD3

```c
float  : TEXCOORD0
float2 : TEXCOORD0
float3 : TEXCOORD0
float4 : TEXCOORD0
```

The same rules apply to:

```text
TEXCOORD1
TEXCOORD2
TEXCOORD3
```

---

### COLOR0 - COLOR1

```c
float3 : COLOR0
float4 : COLOR0
```

The same rules apply to:

```text
COLOR1
```

---

### BLENDINDICES

```c
uint4 : BLENDINDICES
```

---

### BLENDWEIGHT

```c
float4 : BLENDWEIGHT
```

---

# Math Functions

## Angle Conversion

```c
float radians(float degrees)
float degrees(float radians)
```

---

## Trigonometry

```c
float sin(float x)
float cos(float x)
float tan(float x)

float asin(float x)
float acos(float x)

float atan(float x)
float atan(float y, float x)

float sinh(float x)
float cosh(float x)
float tanh(float x)

float asinh(float x)
float acosh(float x)
float atanh(float x)
```

---

## Exponential

```c
float pow(float x, float y)

float exp(float x)
float exp2(float x)

float log(float x)
float log2(float x)

float sqrt(float x)
float inversesqrt(float x)
```

---

## Common

```c
float abs(float x)

float sign(float x)

float floor(float x)
float ceil(float x)
float trunc(float x)

float round(float x)
float roundEven(float x)

float fract(float x)

float mod(float x, float y)

float min(float x, float y)
float max(float x, float y)

float clamp(
    float x,
    float minValue,
    float maxValue)

float mix(
    float a,
    float b,
    float t)

float step(
    float edge,
    float x)

float smoothstep(
    float minValue,
    float maxValue,
    float x)

float fma(
    float a,
    float b,
    float c)
```

---

## Geometry

```c
float length(float2 v)
float length(float3 v)
float length(float4 v)

float distance(float2 a, float2 b)
float distance(float3 a, float3 b)
float distance(float4 a, float4 b)

float dot(float2 a, float2 b)
float dot(float3 a, float3 b)
float dot(float4 a, float4 b)

float3 cross(float3 a, float3 b)

float2 normalize(float2 v)
float3 normalize(float3 v)
float4 normalize(float4 v)

float3 faceforward(
    float3 N,
    float3 I,
    float3 Nref)

float3 reflect(
    float3 I,
    float3 N)

float3 refract(
    float3 I,
    float3 N,
    float eta)
```

---

## Matrix

```c
float2x2 transpose(float2x2 m)
float3x3 transpose(float3x3 m)
float4x4 transpose(float4x4 m)

float determinant(float2x2 m)
float determinant(float3x3 m)
float determinant(float4x4 m)

float2x2 inverse(float2x2 m)
float3x3 inverse(float3x3 m)
float4x4 inverse(float4x4 m)

float3x3 outerProduct(
    float3 a,
    float3 b)
```

---

## Derivatives

```c
float dFdx(float x)
float dFdy(float x)

float fwidth(float x)
```

---

## Texture Sampling

```c
float4 texture(
    Texture2D tex,
    float2 uv)

float4 textureLod(
    Texture2D tex,
    float2 uv,
    float lod)

float4 textureGrad(
    Texture2D tex,
    float2 uv,
    float2 ddx,
    float2 ddy)

float4 texelFetch(
    Texture2D tex,
    int2 coord,
    int lod)
```

# Comments

ForgeShader supports both single-line and multi-line comments.

Comments are ignored by the compiler and do not affect runtime performance.

---

## Single-Line Comments

Use `//` to comment the remainder of the current line.

```c
// This is a comment

float value = 5.0f;

// Inline comment
float roughness = 0.5f; // Material roughness
```

---

## Multi-Line Comments

Use `/* */` to comment multiple lines.

```c
/*
This is a multi-line comment.

It can span
multiple lines.
*/

float value = 5.0f;
```

---

## Documentation Comments

Use `/** */` to document shaders, structures, variables, and functions.

Documentation comments may be consumed by editor tooling, reflection systems, and generated documentation.

```c
/**
 * Base color of the material.
 */
float3 BaseColor;

/**
 * Surface roughness.
 * Expected range: 0.0 - 1.0
 */
float Roughness;
```

```c
/**
 * Computes the final fragment color.
 */
FragOut frag(FragIn input)
{
    ...
}
```

---

## Nested Comments

Nested block comments are not supported.

Invalid:

```c
/*
Outer comment

    /*
    Inner comment
    */

*/
```

---

## Comment Placement

Comments may appear before declarations, inside structures, or inline with code.

```c
// Material properties
Properties
{
    float3 BaseColor;
    float Roughness;
}

/* Albedo texture */
Texture2D Albedo;
```
