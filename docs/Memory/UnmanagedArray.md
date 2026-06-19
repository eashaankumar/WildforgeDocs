# UnmanagedArray<T>

`UnmanagedArray<T>` is a fixed-size unmanaged contiguous array for storing unmanaged values in native memory.

Unlike `UnmanagedList<T>`, the array has a fixed length that cannot grow or shrink after construction.

---

## Declaration

```csharp
public unsafe struct UnmanagedArray<T> : IDisposable
    where T : unmanaged
```

---

## Purpose

`UnmanagedArray<T>` is designed for situations where the number of elements is known ahead of time.

Typical uses include:

- Lookup tables
- Chunk data
- Fixed-size simulation buffers
- Grid storage
- Per-thread scratch memory
- Entity component arrays

---

## Construction

```csharp
var array = new UnmanagedArray<int>(1024);
```

Creates an unmanaged array containing 1024 elements.

All elements are initialized to their default value.

---

## Properties

### Ptr

Pointer to the first element.

```csharp
int* ptr = array.Ptr;
```

---

### Length

Number of elements contained in the array.

```csharp
int length = array.Length;
```

---

### IsCreated

Returns whether unmanaged memory has been allocated.

```csharp
if (array.IsCreated)
{
}
```

---

## Element Access

### TryGet()

Safely retrieves a pointer to an element.

```csharp
if (array.TryGet(index, out var ptr))
{
    *ptr = 5;
}
```

Returns `false` if the index is out of range.

---

### TryGetValue()

Safely retrieves a copy of an element.

```csharp
if (array.TryGetValue(index, out var value))
{
}
```

Returns `false` if the index is invalid.

---

### TrySet()

Safely writes an element.

```csharp
array.TrySet(index, value);
```

Returns `false` if the index is invalid.

---

### UnsafeElementAt()

Returns a reference to an element without bounds checking.

```csharp
ref var value = ref array.UnsafeElementAt(index);
```

Only use when the index is known to be valid.

---

## Views

### AsSpan()

Creates a managed span over the unmanaged memory.

```csharp
Span<int> span = array.AsSpan();
```

---

### AsView()

Creates an unmanaged view over the array.

```csharp
UnmanagedView<int> view = array.AsView();
```

---

## Copying

### Copy()

Copies a range of elements between unmanaged arrays.

```csharp
UnmanagedArray<int>.Copy(
    source,
    sourceIndex,
    destination,
    destinationIndex,
    length);
```

The source and destination ranges must both be valid or an `ArgumentOutOfRangeException` is thrown.

This operation performs a direct memory copy and is significantly faster than copying elements individually.

---

## Memory Operations

### Clear()

Sets every element to its default value.

```csharp
array.Clear();
```

---

## Disposal

The unmanaged memory must be released when the array is no longer needed.

```csharp
array.Dispose();
```

---

## Performance

| Operation | Complexity |
|-----------|-----------:|
| Constructor | O(n) |
| TryGet | O(1) |
| TryGetValue | O(1) |
| TrySet | O(1) |
| UnsafeElementAt | O(1) |
| AsSpan | O(1) |
| AsView | O(1) |
| Copy | O(n) |
| Clear | O(n) |
| Dispose | O(1) |

---

## Design

`UnmanagedArray<T>` provides:

- Fixed-size allocation.
- Contiguous unmanaged memory.
- Zero managed allocations.
- Pointer-based access.
- Span support.
- View support.
- Fast native memory copying.
- Compatibility with `where T : unmanaged`.
- Suitable for Burst-compatible and high-performance systems.

---

## Comparison

| Feature | UnmanagedArray | UnmanagedList |
|---------|----------------|---------------|
| Fixed size | ✅ | ❌ |
| Dynamic growth | ❌ | ✅ |
| Pointer access | ✅ | ✅ |
| Span support | ✅ | ✅ |
| View support | ✅ | ✅ |
| Fast memory copy | ✅ | ❌ |
| Requires Dispose | ✅ | ✅ |

---

## Example

```csharp
using var source = new UnmanagedArray<float>(256);
using var destination = new UnmanagedArray<float>(256);

for (var i = 0; i < source.Length; i++)
{
    source.TrySet(i, i * 0.5f);
}

UnmanagedArray<float>.Copy(
    source,
    0,
    destination,
    0,
    source.Length);

Span<float> span = destination.AsSpan();

span[0] = 10;
```