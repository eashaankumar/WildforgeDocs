# UnmanagedDictionary<TKey, TValue>

A high-performance unmanaged hash table for storing key-value pairs without managed allocations.

`UnmanagedDictionary<TKey, TValue>` stores all data in unmanaged memory, making it suitable for performance-critical systems such as ECS, simulation, serialization, and native collections.

Both `TKey` and `TValue` must be unmanaged types.

---

## Declaration

```csharp
public unsafe struct UnmanagedDictionary<TKey, TValue> : IDisposable
    where TKey : unmanaged, IEquatable<TKey>
    where TValue : unmanaged
```

---

## Constructors

### UnmanagedDictionary(int capacity)

Creates a dictionary with the specified initial capacity.

```csharp
var dict = new UnmanagedDictionary<int, float>(64);
```

---

## Properties

### Count

Number of stored key/value pairs.

```csharp
int count = dict.Count;
```

### Capacity

Current number of buckets.

```csharp
int capacity = dict.Capacity;
```

---

## Methods

### TryAdd()

Attempts to add a key/value pair.

Returns `false` if the key already exists.

```csharp
dict.TryAdd(id, entity);
```

---

### TryRemove()

Removes a key.

Returns whether the key existed.

```csharp
dict.TryRemove(id);
```

---

### TryGetValue()

Retrieves the value associated with a key.

```csharp
if (dict.TryGetValue(id, out var value))
{
    // use value
}
```

---

### ContainsKey()

Checks whether a key exists.

```csharp
if (dict.ContainsKey(id))
{
}
```

---

### Clear()

Removes all entries while keeping allocated memory.

```csharp
dict.Clear();
```

---

### Dispose()

Releases unmanaged memory.

```csharp
dict.Dispose();
```

---

## Performance

Average-case operations:

| Operation | Complexity |
|------------|-----------:|
| Lookup | O(1) |
| Insert | O(1) |
| Remove | O(1) |
| Resize | O(n) |

---

## Notes

- Stores data entirely in unmanaged memory.
- Uses open addressing.
- Automatically resizes as needed.
- Removing entries leaves tombstones internally.
- Keys must implement `IEquatable<TKey>`.