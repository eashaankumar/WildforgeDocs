# UnmanagedString64

A fixed-capacity unmanaged UTF-8 string that can be embedded directly inside unmanaged structs.

`UnmanagedString64` stores its bytes inline without allocating managed memory, making it suitable for `TypedEntityPools`, unmanaged collections, Burst jobs, and native memory.

---

## Declaration

```csharp
public unsafe struct UnmanagedString64 :
    IEquatable<UnmanagedString64>
```

---

## Capacity

Stores up to **63 UTF-8 bytes**. Struct total size is `64 bytes`.

The `Length` field is stored separately and does not reduce the available storage capacity.

ASCII text can store up to 64 characters. Characters requiring multiple UTF-8 bytes (such as emoji or many non-Latin scripts) reduce the maximum number of characters that can be stored.

---

## Properties

### Length

The number of UTF-8 bytes currently stored.

```csharp
byte length = value.Length;
```

---

## Methods

### TrySet()

Attempts to copy a managed string into the unmanaged buffer.

Returns `false` if the UTF-8 encoded string exceeds the available capacity.

```csharp
UnmanagedString64 name = default;

if (name.TrySet("Player"))
{
    // Success
}
```

---

### Clear()

Removes all stored characters.

```csharp
name.Clear();
```

---

### Equals()

Determines whether two unmanaged strings contain identical UTF-8 data.

```csharp
if (name.Equals(other))
{
}
```

---

### ToString()

Creates a managed `string` from the stored UTF-8 bytes.

```csharp
string text = name.ToString();
```

---

### GetHashCode()

Returns a hash code for the stored UTF-8 bytes.

Suitable for hash-based collections.

---

## Performance

| Operation | Complexity |
|-----------|-----------:|
| TrySet | O(n) |
| Equals | O(n) |
| ToString | O(n) |
| GetHashCode | O(n) |
| Clear | O(1) |

---

## Remarks

- Stores UTF-8 bytes inline using a fixed buffer.
- Allocates no managed memory while stored.
- Can be embedded directly inside unmanaged structs.
- Compatible with `where T : unmanaged`.
- Requires no disposal.
- Supports value copying.
- Equality compares the UTF-8 byte sequence.
- `ToString()` allocates a managed `string`.
```