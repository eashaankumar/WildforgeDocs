# FastNoiseLite

## Overview

`FastNoiseLite` is the CPU-side C# 9.0 port of the HLSL FastNoiseLite implementation used by the terrain generation
shaders.

The goal is deterministic CPU/GPU parity:

```text
same seed + same position + same settings = same noise definition
```

Namespace:

```csharp
namespace Forgewild.Core.Math;
```

Primary type:

```csharp
FastNoiseLite
```

Primary usage:

```csharp
var noise = new FastNoiseLite(seed);

float n2 = noise.GetNoise2D(x, y);
float n3 = noise.GetNoise3D(x, y, z);

noise.DomainWarp2D(ref x, ref y);
noise.DomainWarp3D(ref x, ref y, ref z);
```

The public API is now normal C# class-style usage. The old public `fnl_*` state/functions and raw `FNL_*` constants
should no longer be used from gameplay or generation code.

---

# Basic Usage

## Create Default Noise

```csharp
using Forgewild.Core.Math;

var noise = new FastNoiseLite(1337);
```

Default settings:

```text
Seed = 1337
Frequency = 0.01
Type = OpenSimplex2
Rotation3D = None
Fractal = None
Octaves = 3
Lacunarity = 2.0
Gain = 0.5
WeightedStrength = 0.0
PingPongStrength = 2.0
CellularDistance = EuclideanSq
CellularReturn = Distance
CellularJitter = 1.0
DomainWarp = OpenSimplex2
DomainWarpAmplitude = 30.0
```

---

# Public Settings

```csharp
noise.Seed = 1337;
noise.Frequency = 0.01f;
noise.Type = FastNoiseLite.NoiseType.OpenSimplex2;
noise.Rotation3D = FastNoiseLite.RotationType3D.None;
noise.Fractal = FastNoiseLite.FractalType.None;
noise.Octaves = 3;
noise.Lacunarity = 2.0f;
noise.Gain = 0.5f;
noise.WeightedStrength = 0.0f;
noise.PingPongStrength = 2.0f;
noise.CellularDistance = FastNoiseLite.CellularDistanceFunction.EuclideanSq;
noise.CellularReturn = FastNoiseLite.CellularReturnType.Distance;
noise.CellularJitter = 1.0f;
noise.DomainWarp = FastNoiseLite.DomainWarpType.OpenSimplex2;
noise.DomainWarpAmplitude = 30.0f;
```

---

# 2D Noise

Use 2D noise for:

- heightmaps
- moisture maps
- temperature maps
- biome masks
- foliage density masks
- broad feature fields

```csharp
var noise = new FastNoiseLite(12345);
noise.Frequency = 0.005f;

float x = 100f;
float y = 250f;

float value = noise.GetNoise2D(x, y);
```

Output range is generally:

```text
-1 to 1
```

---

# 3D Noise

Use 3D noise for:

- density fields
- caves
- overhangs
- volumetric terrain
- cloud density
- 3D biome fields

```csharp
var noise = new FastNoiseLite(12345);
noise.Frequency = 0.01f;

float x = 100f;
float y = 40f;
float z = -250f;

float value = noise.GetNoise3D(x, y, z);
```

---

# OpenSimplex2

OpenSimplex2 is the default noise type.

```csharp
var noise = new FastNoiseLite(1337);
noise.Type = FastNoiseLite.NoiseType.OpenSimplex2;
noise.Frequency = 0.01f;

float value = noise.GetNoise3D(worldX, worldY, worldZ);
```

Good for:

- natural terrain variation
- broad terrain density
- smooth organic fields

---

# OpenSimplex2S

```csharp
var noise = new FastNoiseLite(1337);
noise.Type = FastNoiseLite.NoiseType.OpenSimplex2S;
noise.Frequency = 0.01f;

float value = noise.GetNoise3D(worldX, worldY, worldZ);
```

Good for:

- smoother simplex variation
- softer volumetric terrain fields
- less harsh directional artifacts

---

# Perlin Noise

```csharp
var noise = new FastNoiseLite(1337);
noise.Type = FastNoiseLite.NoiseType.Perlin;
noise.Frequency = 0.01f;

float value = noise.GetNoise3D(worldX, worldY, worldZ);
```

Good for:

- classic terrain noise
- soft masks
- legacy-style procedural fields

---

# Value Noise

```csharp
var noise = new FastNoiseLite(1337);
noise.Type = FastNoiseLite.NoiseType.Value;
noise.Frequency = 0.01f;

float value = noise.GetNoise3D(worldX, worldY, worldZ);
```

Good for:

- cheap soft random fields
- coarse procedural variation
- non-gradient masks

---

# Value Cubic Noise

```csharp
var noise = new FastNoiseLite(1337);
noise.Type = FastNoiseLite.NoiseType.ValueCubic;
noise.Frequency = 0.01f;

float value = noise.GetNoise3D(worldX, worldY, worldZ);
```

Good for:

- smoother value noise
- soft feature masks
- low-frequency region variation

---

# FBm Fractal Noise

FBm combines multiple octaves of noise.

```csharp
var noise = new FastNoiseLite(1337);
noise.Type = FastNoiseLite.NoiseType.OpenSimplex2;
noise.Fractal = FastNoiseLite.FractalType.FBM;
noise.Frequency = 0.005f;
noise.Octaves = 5;
noise.Lacunarity = 2.0f;
noise.Gain = 0.5f;

float value = noise.GetNoise3D(worldX, worldY, worldZ);
```

Good for:

- layered terrain detail
- natural height variation
- terrain density variation

---

# Ridged Fractal Noise

Ridged noise creates sharp ridge-like formations.

```csharp
var noise = new FastNoiseLite(1337);
noise.Type = FastNoiseLite.NoiseType.OpenSimplex2;
noise.Fractal = FastNoiseLite.FractalType.Ridged;
noise.Frequency = 0.004f;
noise.Octaves = 5;
noise.Lacunarity = 2.0f;
noise.Gain = 0.5f;

float value = noise.GetNoise3D(worldX, worldY, worldZ);
```

Good for:

- mountain ridges
- canyon walls
- eroded-looking terrain masks

---

# Ping Pong Fractal Noise

```csharp
var noise = new FastNoiseLite(1337);
noise.Type = FastNoiseLite.NoiseType.OpenSimplex2;
noise.Fractal = FastNoiseLite.FractalType.PingPong;
noise.Frequency = 0.005f;
noise.Octaves = 4;
noise.Lacunarity = 2.0f;
noise.Gain = 0.5f;
noise.PingPongStrength = 2.0f;

float value = noise.GetNoise3D(worldX, worldY, worldZ);
```

Good for:

- strange organic patterns
- alien terrain
- stylized density masks

---

# Cellular Noise

Cellular noise is useful for regions, cracks, cells, and biome-like partitions.

```csharp
var noise = new FastNoiseLite(1337);
noise.Type = FastNoiseLite.NoiseType.Cellular;
noise.Frequency = 0.01f;
noise.CellularDistance = FastNoiseLite.CellularDistanceFunction.EuclideanSq;
noise.CellularReturn = FastNoiseLite.CellularReturnType.Distance;
noise.CellularJitter = 1.0f;

float value = noise.GetNoise2D(worldX, worldZ);
```

---

# Cellular Distance Functions

## Euclidean

```csharp
noise.CellularDistance = FastNoiseLite.CellularDistanceFunction.Euclidean;
```

## Euclidean Squared

```csharp
noise.CellularDistance = FastNoiseLite.CellularDistanceFunction.EuclideanSq;
```

## Manhattan

```csharp
noise.CellularDistance = FastNoiseLite.CellularDistanceFunction.Manhattan;
```

## Hybrid

```csharp
noise.CellularDistance = FastNoiseLite.CellularDistanceFunction.Hybrid;
```

---

# Cellular Return Types

```csharp
noise.CellularReturn = FastNoiseLite.CellularReturnType.CellValue;
```

```csharp
noise.CellularReturn = FastNoiseLite.CellularReturnType.Distance;
```

```csharp
noise.CellularReturn = FastNoiseLite.CellularReturnType.Distance2;
```

```csharp
noise.CellularReturn = FastNoiseLite.CellularReturnType.Distance2Add;
```

```csharp
noise.CellularReturn = FastNoiseLite.CellularReturnType.Distance2Sub;
```

```csharp
noise.CellularReturn = FastNoiseLite.CellularReturnType.Distance2Mul;
```

```csharp
noise.CellularReturn = FastNoiseLite.CellularReturnType.Distance2Div;
```

---

# 2D Domain Warp

Domain warp modifies the input coordinates before sampling noise.

```csharp
var warp = new FastNoiseLite(1337);
warp.DomainWarp = FastNoiseLite.DomainWarpType.OpenSimplex2;
warp.DomainWarpAmplitude = 30f;
warp.Frequency = 0.01f;

float x = worldX;
float y = worldZ;

warp.DomainWarp2D(ref x, ref y);

float value = warp.GetNoise2D(x, y);
```

Good for:

- warped biome masks
- meandering terrain patterns
- less grid-like feature fields

---

# 3D Domain Warp

```csharp
var warp = new FastNoiseLite(1337);
warp.DomainWarp = FastNoiseLite.DomainWarpType.OpenSimplex2;
warp.DomainWarpAmplitude = 20f;
warp.Frequency = 0.01f;

float x = worldX;
float y = worldY;
float z = worldZ;

warp.DomainWarp3D(ref x, ref y, ref z);

float value = warp.GetNoise3D(x, y, z);
```

Good for:

- caves
- cloud volumes
- 3D density deformation
- less uniform volumetric terrain

---

# Progressive Domain Warp

```csharp
var warp = new FastNoiseLite(1337);
warp.Fractal = FastNoiseLite.FractalType.DomainWarpProgressive;
warp.DomainWarp = FastNoiseLite.DomainWarpType.OpenSimplex2;
warp.DomainWarpAmplitude = 25f;
warp.Frequency = 0.01f;
warp.Octaves = 3;
warp.Lacunarity = 2.0f;
warp.Gain = 0.5f;

float x = worldX;
float y = worldZ;

warp.DomainWarp2D(ref x, ref y);
```

Progressive warp feeds each warped coordinate into the next octave.

Good for:

- strong organic distortion
- rivers
- caves
- chaotic terrain masks

---

# Independent Domain Warp

```csharp
var warp = new FastNoiseLite(1337);
warp.Fractal = FastNoiseLite.FractalType.DomainWarpIndependent;
warp.DomainWarp = FastNoiseLite.DomainWarpType.OpenSimplex2;
warp.DomainWarpAmplitude = 25f;
warp.Frequency = 0.01f;
warp.Octaves = 3;
warp.Lacunarity = 2.0f;
warp.Gain = 0.5f;

float x = worldX;
float y = worldZ;

warp.DomainWarp2D(ref x, ref y);
```

Independent warp accumulates octave offsets without feeding warped coordinates forward.

Good for:

- more stable layered distortion
- biome masks
- terrain feature masks

---

# 3D Rotation Settings

Rotation settings help reduce directional artifacts when sampling 2D planes from 3D noise.

## No Rotation

```csharp
noise.Rotation3D = FastNoiseLite.RotationType3D.None;
```

## Improve XY Planes

```csharp
noise.Rotation3D = FastNoiseLite.RotationType3D.ImproveXYPlanes;
```

## Improve XZ Planes

```csharp
noise.Rotation3D = FastNoiseLite.RotationType3D.ImproveXZPlanes;
```

For terrain sampled mostly on XZ:

```csharp
noise.Rotation3D = FastNoiseLite.RotationType3D.ImproveXZPlanes;
```

---

# Terrain Height Example

```csharp
var height = new FastNoiseLite(9001);
height.Type = FastNoiseLite.NoiseType.OpenSimplex2;
height.Fractal = FastNoiseLite.FractalType.FBM;
height.Frequency = 0.0025f;
height.Octaves = 5;
height.Lacunarity = 2.0f;
height.Gain = 0.5f;

float n = height.GetNoise2D(worldX, worldZ);
float terrainHeight = n * 200f;
```

---

# Terrain Density Example

```csharp
var density = new FastNoiseLite(9001);
density.Type = FastNoiseLite.NoiseType.OpenSimplex2;
density.Fractal = FastNoiseLite.FractalType.FBM;
density.Frequency = 0.01f;
density.Octaves = 4;
density.Lacunarity = 2.0f;
density.Gain = 0.5f;

float terrainHeight = 120f;

float n = density.GetNoise3D(worldX, worldY, worldZ);
float signedDensity = terrainHeight - worldY + n * 40f;
```

---

# Biome Mask Example

```csharp
var moisture = new FastNoiseLite(100);
moisture.Type = FastNoiseLite.NoiseType.OpenSimplex2;
moisture.Fractal = FastNoiseLite.FractalType.FBM;
moisture.Frequency = 0.0015f;
moisture.Octaves = 4;

float m = moisture.GetNoise2D(worldX, worldZ);
bool wetBiome = m > 0.25f;
```

---

# CPU/GPU Parity Rules

To keep CPU and HLSL outputs aligned:

1. Use the same seed.
2. Use the same frequency.
3. Use the same noise type.
4. Use the same fractal settings.
5. Use the same position values.
6. Do not remap or round coordinates differently on CPU and GPU.
7. Avoid changing float operation ordering in only one implementation.
8. Keep all integer hash math in unchecked wraparound mode on CPU.

---

# Recommended Forgewild Usage Pattern

Create reusable configured noise objects:

```csharp
public sealed class TerrainNoiseSet
{
    public FastNoiseLite Height { get; } = new FastNoiseLite(1000);
    public FastNoiseLite Moisture { get; } = new FastNoiseLite(2000);
    public FastNoiseLite Temperature { get; } = new FastNoiseLite(3000);
    public FastNoiseLite Density { get; } = new FastNoiseLite(4000);
}
```

Initialize once:

```csharp
var noise = new TerrainNoiseSet();

noise.Height.Type = FastNoiseLite.NoiseType.OpenSimplex2;
noise.Height.Fractal = FastNoiseLite.FractalType.FBM;
noise.Height.Frequency = 0.0025f;
noise.Height.Octaves = 5;

noise.Moisture.Frequency = 0.0015f;
noise.Temperature.Frequency = 0.001f;
noise.Density.Frequency = 0.01f;
```

Sample:

```csharp
float height = noise.Height.GetNoise2D(worldX, worldZ);
float moisture = noise.Moisture.GetNoise2D(worldX, worldZ);
float density = noise.Density.GetNoise3D(worldX, worldY, worldZ);
```

---

# Performance Notes

- `FastNoiseLite` is now a regular class with mutable settings.
- Sampling creates the internal state from the current properties, preserving the original math path.
- Create and configure noise objects once, then reuse them.
- Avoid allocating arrays when sampling terrain.
- Sample into preallocated buffers.
- Use deterministic world coordinates.
- SIMD batching can be added later, but may change floating-point operation ordering.
- For CPU/GPU parity, scalar line-equivalent sampling is safer than aggressive SIMD rewrites.
