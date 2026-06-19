# IndexUtilities

`IndexUtilities` provides fast conversion methods between 3D voxel coordinates and linear array indices.

These methods are intended for dense voxel volumes, chunk storage, and any data stored in a one-dimensional array that logically represents a 3D grid.

---

## Declaration

```csharp
public static class IndexUtilities
```

---

## Memory Layout

`IndexUtilities` uses the following row-major memory layout:

```text
index = z * width * height
      + y * width
      + x
```

Coordinate traversal order:

```text
X changes fastest
Y changes next
Z changes slowest
```

For a volume with dimensions:

```text
Width  = X
Height = Y
Depth  = Z
```

the memory is arranged like:

```text
Layer Z = 0

(0,0,0)
(1,0,0)
(2,0,0)
...

then

(0,1,0)
(1,1,0)
...

then

Z = 1
```

This is the standard layout used by most voxel engines.

---

## Methods

### XyzToIndex(Vector3Int)

Converts a voxel position into a linear array index.

```csharp
int index = IndexUtilities.XyzToIndex(
    position,
    width,
    height);
```

---

### XyzToIndex(int x, int y, int z)

Converts individual coordinates into a linear array index.

```csharp
int index = IndexUtilities.XyzToIndex(
    x,
    y,
    z,
    width,
    height);
```

Equivalent formula:

```text
index = z * width * height
      + y * width
      + x
```

---

### IndexToXyz()

Converts a linear array index back into voxel coordinates.

```csharp
Vector3Int xyz =
    IndexUtilities.IndexToXyz(
        index,
        width,
        height);
```

---

## Example

```csharp
const int width = 16;
const int height = 16;
const int depth = 16;

var position = new Vector3Int(4, 7, 12);

int index = IndexUtilities.XyzToIndex(
    position,
    width,
    height);

Vector3Int recovered =
    IndexUtilities.IndexToXyz(
        index,
        width,
        height);
```

After conversion:

```text
recovered == position
```

---

## Typical Usage

`IndexUtilities` is commonly used for:

- Chunk voxel storage
- Terrain density fields
- Light maps
- Navigation grids
- Cellular automata
- Sparse-to-dense indexing
- Any 3D data stored in a one-dimensional array

---

## Performance

| Operation | Complexity |
|-----------|-----------:|
| XyzToIndex | O(1) |
| IndexToXyz | O(1) |

Both methods are marked with `MethodImplOptions.AggressiveInlining` and perform only integer arithmetic, making them suitable for hot loops in voxel meshing, terrain generation, and simulation systems.

---

## Design

`IndexUtilities` is:

- Allocation-free
- Branch-free
- Deterministic
- Compatible with unmanaged data structures
- Suitable for high-performance voxel processing

It is designed to work seamlessly with containers such as:

- `UnmanagedArray<T>`
- `UnmanagedList<T>`
- `VoxelDataVolume<T>`
- Chunk-based storage systems
```