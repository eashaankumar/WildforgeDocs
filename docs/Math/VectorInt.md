# Vector2Int & Vector3Int

`Vector2Int` and `Vector3Int` are lightweight integer vector types used throughout Wildforge for grid-based mathematics.

Unlike `Vector2` and `Vector3`, these types store integer coordinates and are intended for chunk positions, voxel coordinates, tile locations, and other discrete spatial calculations.

---

## Declaration

```csharp
public struct Vector2Int
```

```csharp
public struct Vector3Int
```

---

## Purpose

These types provide:

- Integer-based coordinates
- Compact memory layout
- Fast arithmetic
- Equality comparisons
- Utility functions
- Sequential unmanaged layout

Typical uses include:

- Chunk coordinates
- Voxel indices
- Tile maps
- Grid navigation
- Array indexing
- Spatial hashing

---

## Memory Layout

### Vector2Int

```text
+-----+-----+
|  X  |  Y  |
+-----+-----+
```

Size: **8 bytes**

---

### Vector3Int

```text
+-----+-----+-----+
|  X  |  Y  |  Z  |
+-----+-----+-----+
```

Size: **12 bytes**

Both structs use sequential layout for efficient storage inside unmanaged containers and entity pools.

---

## Construction

### Vector2Int

```csharp
var a = new Vector2Int(5, 8);

var b = new Vector2Int(10);
```

Creates:

```
(5, 8)

(10, 10)
```

---

### Vector3Int

```csharp
var a = new Vector3Int(1, 2, 3);

var b = new Vector3Int(7);
```

Creates:

```
(1, 2, 3)

(7, 7, 7)
```

---

## Static Values

### Vector2Int

```csharp
Vector2Int.Zero
Vector2Int.One

Vector2Int.Up
Vector2Int.Down
Vector2Int.Left
Vector2Int.Right
```

---

### Vector3Int

```csharp
Vector3Int.Zero
Vector3Int.One

Vector3Int.Up
Vector3Int.Down

Vector3Int.Left
Vector3Int.Right

Vector3Int.Forward
Vector3Int.Back
```

---

## Arithmetic

Both types support:

```csharp
+
-
*
/
```

Example:

```csharp
var c = a + b;

var d = a * 5;
```

---

## Dot Product

```csharp
int dot = Vector3Int.Dot(a, b);
```

```csharp
int dot = Vector2Int.Dot(a, b);
```

---

## Cross Product

`Vector3Int` supports the cross product.

```csharp
Vector3Int normal = Vector3Int.Cross(a, b);
```

---

## LengthSquared

Returns the squared length without computing a square root.

```csharp
int length2 = vector.LengthSquared();
```

Useful for comparisons.

---

## Min / Max

```csharp
Vector3Int.Min(a, b);

Vector3Int.Max(a, b);
```

```csharp
Vector2Int.Min(a, b);

Vector2Int.Max(a, b);
```

---

## Clamp

```csharp
Vector3Int.Clamp(value, min, max);

Vector2Int.Clamp(value, min, max);
```

---

## Absolute Value

```csharp
Vector3Int.Abs(vector);

Vector2Int.Abs(vector);
```

---

## Equality

```csharp
if (a == b)
{
}

if (a != b)
{
}
```

Supports:

- `==`
- `!=`
- `Equals`
- `GetHashCode`

---

## Conversions

### Vector2Int

```csharp
Vector2 vector = (Vector2)grid;

Vector3 vector3 = (Vector3)grid;
```

---

### Vector3Int

```csharp
Vector3 vector = (Vector3)grid;
```

---

## Performance

| Operation | Complexity |
|-----------|-----------:|
| Construction | O(1) |
| Addition | O(1) |
| Subtraction | O(1) |
| Multiplication | O(1) |
| Division | O(1) |
| Dot | O(1) |
| Cross | O(1) |
| Min | O(1) |
| Max | O(1) |
| Clamp | O(1) |
| Abs | O(1) |
| Equals | O(1) |
| GetHashCode | O(1) |

---

## Design

Unlike `System.Numerics.Vector<T>`, these types intentionally store individual fields:

```csharp
public int X;
public int Y;
public int Z;
```

instead of SIMD vectors.

This design provides:

- Compact memory footprint
- Better cache efficiency
- Stable 8-byte and 12-byte layouts
- Mutable fields
- Excellent interoperability with unmanaged memory
- Compatibility with entity pools and serialization

The .NET JIT can still generate SIMD instructions for many arithmetic operations involving these types, while avoiding the larger memory footprint and hardware-dependent size of `Vector<T>`.

---

## Example

```csharp
var chunk = new Vector3Int(5, 2, -3);

var local = new Vector3Int(12, 4, 8);

var world = chunk * 16 + local;

if (world == Vector3Int.Zero)
{
    // Origin
}
```