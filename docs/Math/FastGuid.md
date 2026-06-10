# FastGuid

A lightweight, high-performance 128-bit identifier for C# projects.

`FastGuid` is designed for game engines, ECS systems, asset databases, scene graphs, render pipelines, and other performance-sensitive code where millions of identifier comparisons and dictionary lookups may occur.

Unlike `System.Guid`, `FastGuid` exposes its internal representation as two `ulong` values, enabling extremely fast equality checks, hashing, sorting, and serialization.

---

## Features

- 128-bit unique identifier
- Allocation-free comparisons
- Fast dictionary and hash set lookups
- Value type (`struct`)
- Binary serialization friendly
- Deterministic ordering support
- Compatible with existing `System.Guid`
- C# 9 compatible

---

## Basic Usage

### Creating a New ID

```csharp
FastGuid id = FastGuid.NewGuid();
```

### Checking for Empty

```csharp
if (id == FastGuid.Empty)
{
    // Invalid ID
}
```

### Equality

```csharp
FastGuid a = FastGuid.NewGuid();
FastGuid b = FastGuid.NewGuid();

bool equal = a == b;
```

---

## Dictionary Usage

`FastGuid` is intended to be used as a key in collections.

```csharp
Dictionary<FastGuid, Entity> entities = new();

FastGuid id = FastGuid.NewGuid();

entities[id] = entity;

if (entities.TryGetValue(id, out Entity result))
{
    // Found
}
```

---

## HashSet Usage

```csharp
HashSet<FastGuid> loadedAssets = new();

loadedAssets.Add(assetId);

bool exists = loadedAssets.Contains(assetId);
```

---

## Sorting

Because `FastGuid` implements `IComparable<FastGuid>`, collections may be sorted.

```csharp
List<FastGuid> ids = GetIds();

ids.Sort();
```

---

## Conversion from System.Guid

### From Guid

```csharp
Guid guid = Guid.NewGuid();

FastGuid id = FastGuid.FromGuid(guid);
```

### To Guid

```csharp
Guid guid = id.ToGuid();
```

---

## String Serialization

### Convert to String

```csharp
string text = id.ToString();
```

Example output:

```text
d0fa90ef4d364d8892d0d4b3b8e4876f
```

### Parse

```csharp
FastGuid id = FastGuid.Parse(text);
```

### TryParse

```csharp
if (FastGuid.TryParse(text, out FastGuid id))
{
    // Success
}
```

---

## Binary Serialization

### Write

```csharp
writer.Write(id.A);
writer.Write(id.B);
```

### Read

```csharp
ulong a = reader.ReadUInt64();
ulong b = reader.ReadUInt64();

FastGuid id = new FastGuid(a, b);
```

This is typically faster than serializing through strings.

---

## Internal Layout

`FastGuid` stores data as:

```csharp
struct FastGuid
{
    ulong A;
    ulong B;
}
```

Memory size:

```text
16 bytes
```

Same size as `System.Guid`.

---

## Performance Characteristics

### Equality

```csharp
return A == other.A &&
       B == other.B;
```

Only two 64-bit comparisons.

### Hashing

Uses a MurmurHash-inspired finalization step to produce high-quality dictionary hashes.

### Dictionary Lookups

Expected performance benefits compared to `Guid`:

| Operation | Guid | FastGuid |
|------------|---------|---------|
| Equality | Fast | Very Fast |
| Hashing | Fast | Very Fast |
| Dictionary Lookup | Fast | Very Fast |
| Serialization | Medium | Fast |
| Binary Save/Load | Medium | Fast |

Actual gains depend on workload and runtime.

---

## Recommended Use Cases

### ECS Entities

```csharp
public struct Entity
{
    public FastGuid Id;
}
```

### Asset References

```csharp
public class Asset
{
    public FastGuid Id;
}
```

### Scene Objects

```csharp
public class SceneNode
{
    public FastGuid Id;
}
```

### Networking

```csharp
packet.Write(id.A);
packet.Write(id.B);
```

### Save Files

```csharp
save.Write(id.A);
save.Write(id.B);
```

---

## Best Practices

### Good

Generate once and reuse:

```csharp
FastGuid id = FastGuid.NewGuid();
```

Store directly:

```csharp
Dictionary<FastGuid, Component> components;
```

Serialize as raw binary:

```csharp
writer.Write(id.A);
writer.Write(id.B);
```

### Avoid

Generating IDs every frame:

```csharp
for (...)
{
    FastGuid.NewGuid();
}
```

Converting to strings during gameplay:

```csharp
string text = id.ToString();
```

Use string conversion primarily for:

- Debugging
- Logging
- Editor tooling
- Save file export

---

## Example Asset Database

```csharp
public sealed class AssetDatabase
{
    private readonly Dictionary<FastGuid, Asset> _assets = new();

    public FastGuid Register(Asset asset)
    {
        FastGuid id = FastGuid.NewGuid();

        _assets[id] = asset;

        return id;
    }

    public bool TryGet(FastGuid id, out Asset asset)
    {
        return _assets.TryGetValue(id, out asset);
    }
}
```

---

## Notes

`FastGuid` does **not** generate IDs faster than `Guid.NewGuid()` because it uses the operating system's secure GUID generator internally.

Its primary purpose is to provide:

- Faster comparisons
- Faster hashing
- Faster serialization
- Cleaner binary storage
- Better integration with performance-critical systems

For most game engine workloads, those operations occur far more frequently than GUID generation itself.