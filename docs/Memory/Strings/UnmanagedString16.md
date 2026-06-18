# UnmanagedString16

A fixed-size unmanaged UTF-8 string occupying **16 bytes** of memory.

`UnmanagedString16` stores its contents inline without allocating managed memory, making it suitable for `TypedEntityPools`, unmanaged collections, Burst jobs, and native memory.

---

## Declaration

```csharp
public unsafe struct UnmanagedString16 :
    IEquatable<UnmanagedString16>
```

---

## Capacity

Stores up to **15 UTF-8 bytes**.

The remaining byte stores the string length.

ASCII text can contain up to **15 characters**.

---

## Properties

### Length

The number of UTF-8 bytes currently stored.

---

## Methods

### TrySet()

Attempts to store a managed string.

Returns `false` if the UTF-8 byte length exceeds capacity.

### Clear()

Removes all stored characters.

### Equals()

Determines whether two unmanaged strings contain identical UTF-8 data.

### ToString()

Creates a managed `string`.

### GetHashCode()

Returns a hash code based on the stored UTF-8 bytes.

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

- Occupies exactly **16 bytes**.
- Stores UTF-8 bytes inline.
- Requires no disposal.
- Can be embedded inside unmanaged structs.
- Compatible with `where T : unmanaged`.