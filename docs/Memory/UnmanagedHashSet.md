# UnmanagedHashSet<T>

An unmanaged hash set for storing unique values without managed allocations.

`UnmanagedHashSet<T>` provides constant-time lookup while allocating entirely in unmanaged memory.

`T` must be unmanaged.

---

## Declaration

```csharp
public unsafe struct UnmanagedHashSet<T> : IDisposable
    where T : unmanaged, IEquatable<T>
```

---

## Constructors

### UnmanagedHashSet(int capacity)

Creates a hash set.

```csharp
var set = new UnmanagedHashSet<int>(64);
```

---

## Properties

### Count

Number of stored values.

```csharp
int count = set.Count;
```

### Capacity

Number of buckets.

```csharp
int capacity = set.Capacity;
```

---

## Methods

### TryAdd()

Attempts to add a value.

Returns `false` if it already exists.

```csharp
set.TryAdd(value);
```

---

### TryRemove()

Removes a value.

```csharp
set.TryRemove(value);
```

---

### Contains()

Determines whether a value exists.

```csharp
if (set.Contains(value))
{
}
```

---

### Clear()

Removes all values while preserving allocated memory.

```csharp
set.Clear();
```

---

### Dispose()

Frees unmanaged memory.

```csharp
set.Dispose();
```

---

## Performance

| Operation | Complexity |
|------------|-----------:|
| Contains | O(1) |
| Add | O(1) |
| Remove | O(1) |
| Resize | O(n) |

---

## Notes

- Unmanaged memory only.
- Stores unique values.
- Uses open addressing.
- Automatically resizes when needed.