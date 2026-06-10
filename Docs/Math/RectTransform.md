# Rect Transform

Struct to hold transform of UI components, or any component that
lives in screen space.

## Makeup

```csharp
Vector3 Position;
Quaternion Rotation;
Vector2 Size;
Vector3 Scale;
Vector2 Pivot01;
```

Note: Pivot is in range [0,1] where:

(0,0) is bottom left corner

(0.5, 0.5) is center

(1,1) is top right corner.

Pivot controls where the element's origin is located. Movement, rotation, and scaling happen around this point.