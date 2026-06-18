# Transform

The Wildforge math library provides several transform types optimized for different use cases.

| Type | Position | Rotation | Scale |
|------|----------|----------|-------|
| `TransformUnscaled` | ✅ | ✅ | ❌ |
| `Transform` | ✅ | ✅ | Uniform (`float`) |
| `TransformScaled` | ✅ | ✅ | Non-uniform (`Vector3`) |
| `RectTransform` | ✅ | ✅ | `Vector3` + UI Size/Pivot |

---

# TransformUnscaled

`TransformUnscaled` stores a 3D position and rotation without scale.

```csharp
public struct TransformUnscaled
{
    public Vector3 Position;
    public Quaternion Rotation;
}
```

## Purpose

Use when scaling is unnecessary.

Typical examples include:

- Cameras
- Lights
- Spawn points
- Physics objects
- Waypoints

## Matrix

```csharp
Matrix4x4 matrix = transform.ToMatrix();
```

Builds a **Rotation → Translation (RT)** matrix.

## Direction Vectors

```csharp
transform.Forward();
transform.Right();
transform.Up();
```

Returns the transform's local forward, right and up vectors.

---

# Transform

`Transform` stores a 3D position, rotation and **uniform** scale.

```csharp
public struct Transform
{
    public Vector3 Position;
    public Quaternion Rotation;
    public float Scale;
}
```

## Purpose

This is the standard transform used throughout Wildforge.

Using a scalar scale reduces memory while supporting the vast majority of game objects.

## Matrix

```csharp
Matrix4x4 matrix = transform.ToMatrix();
```

Builds a **Scale → Rotation → Translation (SRT)** matrix.

## Direction Vectors

```csharp
transform.Forward();
transform.Right();
transform.Up();
```

Returns the transform's local forward, right and up vectors.

---

# TransformScaled

`TransformScaled` stores a 3D position, rotation and **non-uniform** scale.

```csharp
public struct TransformScaled
{
    public Vector3 Position;
    public Quaternion Rotation;
    public Vector3 Scale;
}
```

## Purpose

Use when independent scaling along each axis is required.

Typical examples include:

- Stretched meshes
- Procedural geometry
- Imported models
- Special rendering effects

## Matrix

```csharp
Matrix4x4 matrix = transform.ToMatrix();
```

Builds a **Scale → Rotation → Translation (SRT)** matrix.

## Direction Vectors

```csharp
transform.Forward();
transform.Right();
transform.Up();
```

Returns the transform's local forward, right and up vectors.

---

# RectTransform

`RectTransform` stores the transform of a rectangular UI or sprite element.

```csharp
public struct RectTransform
{
    public Vector3 Position;
    public Quaternion Rotation;
    public Vector2 Size;
    public Vector3 Scale;
    public Vector2 Pivot01;
}
```

## Purpose

`RectTransform` extends a standard transform with rectangle-specific data for UI and 2D rendering.

Unlike the other transform types, it stores both a logical size and a normalized pivot.

## Fields

### Position

World position.

### Rotation

Quaternion orientation.

### Size

Rectangle width and height.

### Scale

Per-axis scaling.

### Pivot01

Normalized pivot.

Common values:

| Pivot | Value |
|-------|-------|
| Bottom Left | `(0, 0)` |
| Center | `(0.5, 0.5)` |
| Top Right | `(1, 1)` |

## Matrix Methods

### ToRenderMatrix()

```csharp
Matrix4x4 matrix = transform.ToRenderMatrix();
```

Builds the rendering matrix.

Applies:

1. Pivot offset
2. Size scaling
3. Object scaling
4. Rotation
5. Translation

### ToTRSMatrix()

```csharp
Matrix4x4 matrix = transform.ToTRSMatrix();
```

Builds a standard **Scale → Rotation → Translation** matrix that ignores the rectangle size and pivot.

## Rectangle Methods

### Rect()

```csharp
Rect2D rect = transform.Rect();
```

Returns the transformed axis-aligned rectangle.

### Rect(Matrix4x4 parentTRS)

```csharp
Rect2D rect = transform.Rect(parentMatrix);
```

Returns the transformed rectangle after applying a parent transform.

## Direction Vectors

```csharp
transform.Forward();
transform.Right();
transform.Up();
```

Returns the transform's local forward, right and up vectors.

---

# Design

All transform types share several design goals:

- Sequential memory layout.
- Compatible with unmanaged memory.
- Suitable for storage inside `TypedEntityPools`.
- Quaternion rotation avoids gimbal lock.
- Public fields maximize performance.
- Aggressively inlined helper methods.

Choose the smallest transform type that satisfies the requirements of your object.

---

# Examples

```csharp
Transform transform;

transform.Position += velocity * deltaTime;

Matrix4x4 matrix = transform.ToMatrix();
```

```csharp
TransformUnscaled camera;

camera.Position += direction;
```

```csharp
TransformScaled mesh;

mesh.Scale = new Vector3(2, 1, 0.5f);
```

```csharp
RectTransform ui;

Rect2D bounds = ui.Rect();
Matrix4x4 renderMatrix = ui.ToRenderMatrix();
```