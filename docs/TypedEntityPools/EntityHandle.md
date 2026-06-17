# EntityHandle

Namespace:

```csharp
using Wildforge.Engine.Core.TypedEntityPools;
```

`EntityHandle` is Wildforge's stable runtime identifier for entities stored inside an `EntityPool<T>`.

```csharp
public readonly struct EntityHandle
{
    public readonly int Id;
    public readonly int Generation;

    public static readonly EntityHandle Invalid = default;

    public bool IsValid => Id > 0;
}
```

Unlike pointers or dense indices, an `EntityHandle` remains valid even if the entity moves within the pool's internal storage.

---

# Purpose

The primary goal of `EntityHandle` is to provide **stable runtime identity**.

Internally, `EntityPool<T>` stores entities in a dense contiguous array for maximum cache efficiency.

As entities are destroyed, the pool uses **swap-remove** to eliminate holes.

For example:

```text
Dense Storage

0  Dino A
1  Dino B
2  Dino C
3  Dino D
```

Destroying **B** results in:

```text
0  Dino A
1  Dino D
2  Dino C
```

The dense index of **D** changed from **3** to **1**.

If gameplay code stored dense indices:

```csharp
int dinoIndex = 3;
```

that reference is now incorrect.

Instead, gameplay stores:

```csharp
EntityHandle handle;
```

The handle remains valid regardless of where the entity moves inside the dense array.

---

# Structure

```csharp
public readonly struct EntityHandle
{
    public readonly int Id;
    public readonly int Generation;
}
```

The handle contains two pieces of information:

- Slot ID
- Generation

Together these uniquely identify one lifetime of one entity.

---

# Id

`Id` identifies a slot inside the pool.

It **does not** identify a dense array index.

Internally the pool may store slots like:

```text
Slot 0
Slot 1
Slot 2
Slot 3
```

Wildforge intentionally exposes:

```text
Handle Id

1
2
3
4
```

instead.

This allows:

```csharp
default(EntityHandle)
```

to become:

```text
Id = 0
Generation = 0
```

which is automatically invalid.

---

# Why Id 0 Is Reserved

Every C# struct defaults to zero.

If slot zero were valid, then code such as:

```csharp
public struct Dino
{
    public EntityHandle Target;
}
```

would accidentally contain a valid reference immediately after construction.

By reserving **Id 0** as invalid:

```csharp
default(EntityHandle).IsValid == false
```

which is exactly what most developers expect.

---

# Generation

The second field is:

```csharp
public readonly int Generation;
```

Generation protects against stale handles.

Suppose:

```text
Create Dino A

Handle:
Id = 5
Generation = 0
```

Later:

```text
Destroy Dino A
```

Later again:

```text
Create Dino B
```

The pool reuses slot 5.

Without generations:

```text
Old Handle

Id = 5
```

would now incorrectly point to Dino B.

Instead:

```text
Slot generation increments
```

Now:

```text
Dino A

Id = 5
Generation = 0
```

```text
Dino B

Id = 5
Generation = 1
```

The pool validates:

```text
Slot generation == Handle generation ?
```

If not:

```text
Handle is stale.
```

---

# Validity

`EntityHandle` exposes:

```csharp
public bool IsValid => Id > 0;
```

This checks only whether the handle is structurally valid.

It does **not** guarantee that the entity still exists.

Always validate against the owning pool:

```csharp
if (pool.IsAlive(handle))
{
}
```

or

```csharp
if (pool.TryGet(handle, out var ptr))
{
}
```

---

# Runtime Lifetime

An `EntityHandle` is valid only while:

- the pool exists
- the entity has not been destroyed
- the generation matches

Handles should never outlive their world.

---

# Runtime vs Persistent Identity

`EntityHandle` is **not** a save-game identifier.

Do not serialize handles.

Instead use a persistent identifier system such as:

```text
Guid

↓

Persistent Registry

↓

EntityRef

↓

EntityHandle
```

During loading, persistent IDs resolve back into runtime handles.

---

# EntityHandle vs Dense Index

Dense indices are temporary.

Handles are stable.

Use dense indices only for:

- iteration
- SIMD
- parallel jobs
- renderer extraction

Example:

```csharp
var view = world.Dinos.AsView();

for (int i = 0; i < view.Count; i++)
{
    ref var dino = ref view[i];
}
```

Never store:

```csharp
int DenseIndex;
```

inside gameplay data.

Store:

```csharp
EntityHandle
```

instead.

---

# EntityHandle vs Pointer

Pointers become invalid whenever memory reallocates.

Handles do not.

Pointers are excellent for:

- temporary hot loops
- job execution
- frame-local algorithms

Handles are for:

- gameplay references
- AI targets
- ownership
- relationships
- runtime identity

---

# EntityHandle vs EntityRef

`EntityHandle` identifies an entity **inside one specific pool**.

It does not know which pool.

If an entity can reference different pools, use:

```csharp
EntityRef
```

which contains:

```text
EntityKind

+

EntityHandle
```

For example:

```csharp
public struct Projectile
{
    public EntityRef Owner;
}
```

The owner could be:

- Dino
- Mammal
- Player
- Plant
- Item

`EntityKind` tells the world which pool to search.

`EntityHandle` tells that pool which entity to retrieve.

---

# Typical Usage

Creating an entity:

```csharp
if (world.Dinos.TryCreate(
    new Dino(),
    out var handle))
{
}
```

Looking up:

```csharp
if (world.Dinos.TryGet(
    handle,
    out var dino))
{
    dino->Health -= 10;
}
```

Destroying:

```csharp
world.Dinos.TryDestroy(handle);
```

Checking existence:

```csharp
if (!world.Dinos.IsAlive(handle))
{
}
```

---

# Performance

`EntityHandle` is intentionally tiny.

```text
sizeof(EntityHandle)

8 bytes
```

(two 32-bit integers)

This makes it inexpensive to:

- copy
- compare
- store in arrays
- embed inside large entity structs
- pass through command buffers

Validation is:

```text
O(1)
```

requiring:

- slot lookup
- alive check
- generation comparison

---

# Design Decisions

## Stable References

Gameplay code should never know where an entity lives inside memory.

Only the pool manages physical storage.

---

## Dense Storage

The handle allows the pool to freely reorder dense storage for maximum cache locality.

---

## Default Safety

Using:

```text
Id = 0
```

as invalid makes default values naturally safe.

This eliminates an entire class of bugs involving uninitialized handles.

---

## No Pool Pointer

The handle deliberately contains no pointer back to the pool.

Reasons:

- smaller
- trivially copyable
- unmanaged
- cache friendly
- engine agnostic

The caller already knows which pool they are working with.

---

## No Managed References

`EntityHandle` is fully unmanaged.

It may safely be stored inside any:

```csharp
where T : unmanaged
```

entity.

---

# Best Practices

## Do

Store runtime relationships using handles.

```csharp
public EntityHandle Parent;
```

Validate through the pool before dereferencing.

Use `EntityRef` for cross-pool references.

Use dense indices only inside temporary iteration.

---

## Don't

Do not serialize handles.

Do not assume IDs are dense indices.

Do not manually construct handles except in low-level engine code and tests.

Do not continue using handles after an entity has been destroyed.

---

# Summary

`EntityHandle` is the foundation of Wildforge's runtime identity system.

It enables:

- stable runtime references
- dense cache-friendly entity storage
- O(1) validation
- stale handle detection
- swap-remove deletion
- fully unmanaged entity structs

Together with `EntityPool<T>`, it provides a high-performance alternative to pointer-based or sparse-index entity systems while remaining simple to reason about and safe to use.