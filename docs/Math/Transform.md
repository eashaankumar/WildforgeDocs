# Transform

`Transform` stores position, rotation and scale for 3D objects.

``` csharp
public struct Transform
{
    public Vector3 Position;
    public Quaternion Rotation;
    public Vector3 Scale;
}
```

## Purpose

Used throughout Wildforge for world objects, cameras, rendering and
simulation.

## Matrix

``` csharp
Matrix4x4 world = transform.ToMatrix();
```

Builds a Scale → Rotation → Translation matrix.

## Direction vectors

``` csharp
transform.Forward();
transform.Right();
transform.Up();
```

## RectTransform

`RectTransform` extends the idea for UI/sprites by adding:

-   Size
-   Pivot01

and methods:

-   ToRenderMatrix()
-   ToTRSMatrix()
-   Rect()

## Design

-   Quaternion rotation avoids gimbal lock.
-   Sequential layout is friendly to unmanaged memory.
-   Public fields maximize performance.
-   Suitable for dense storage inside entity pools.

## Example

``` csharp
transform.Position += velocity * dt;
Matrix4x4 matrix = transform.ToMatrix();
```
