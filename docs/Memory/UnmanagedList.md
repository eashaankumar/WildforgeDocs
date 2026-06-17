# UnmanagedList`<T>`{=html}

Namespace:

``` csharp
using Wildforge.Engine.Core.Memory;
```

`UnmanagedList<T>` is Wildforge's fundamental contiguous unmanaged
container.

It behaves similarly to `List<T>`, but stores data in unmanaged memory
and is designed for high-performance simulation, rendering, physics, AI,
and other data-oriented systems.

``` csharp
public unsafe struct UnmanagedList<T>
    where T : unmanaged
```

------------------------------------------------------------------------

# Philosophy

`UnmanagedList<T>` is the foundation of nearly every low-level Wildforge
container.

Higher-level structures such as:

-   `UnmanagedStack<T>`
-   `EntityPool<T>`
-   future hash maps, queues and arenas

are built on top of it.

Unlike `List<T>`, the container:

-   allocates unmanaged memory
-   never stores managed references
-   is deterministic
-   is suitable for dense engine data
-   exposes pointers for maximum performance

------------------------------------------------------------------------

# TryX vs UnsafeX

Every mutating operation follows one of two styles.

## TryX

Safe APIs validate inputs and return `bool` instead of throwing.

``` csharp
if(list.TryAdd(value))
{
    ...
}
```

## UnsafeX

Unsafe APIs skip validation entirely.

``` csharp
list.UnsafeAdd(value);
```

Use these only when correctness has already been established.

This pattern avoids exception overhead inside hot loops.

------------------------------------------------------------------------

# Construction

``` csharp
var list = new UnmanagedList<int>(128);
```

Capacity specifies the initial allocation.

The list may grow automatically.

------------------------------------------------------------------------

# Adding Elements

``` csharp
list.TryAdd(value);

list.UnsafeAdd(value);
```

Default construction:

``` csharp
list.TryAddDefault(out var ptr);

list.UnsafeAddDefault(out var ptr);
```

Writing directly into returned memory avoids copying large structs.

------------------------------------------------------------------------

# Access

Safe:

``` csharp
list.TryGet(index, out var value);
```

Unsafe:

``` csharp
ref var value = ref list.UnsafeElementAt(index);
```

Pointer access:

``` csharp
T* ptr = list.Ptr;
```

------------------------------------------------------------------------

# Capacity

``` csharp
list.TryEnsureCapacity(1024);

list.TrySetCapacity(2048);

list.TryTrimExcess();
```

These methods never invalidate existing values unless a resize occurs.

------------------------------------------------------------------------

# Removal

Swap-back removal:

``` csharp
list.TryRemoveAtSwapBack(index);

list.UnsafeRemoveAtSwapBack(index);
```

The final element is moved into the removed slot.

Order is **not preserved**.

Time complexity:

``` text
O(1)
```

Ideal for entity containers.

------------------------------------------------------------------------

# Views

``` csharp
UnmanagedView<T> view = list.AsView();
```

A view exposes:

-   pointer
-   length

without exposing ownership or structural mutation.

Views are intended for:

-   jobs
-   algorithms
-   parallel iteration
-   rendering

------------------------------------------------------------------------

# Lifetime

``` csharp
list.Clear();

list.Dispose();
```

`Clear()` removes all logical elements but retains allocated memory.

`Dispose()` releases unmanaged memory.

After disposal the list should not be used again.

------------------------------------------------------------------------

# Performance

  Operation                            Complexity
  ----------------- -----------------------------
  Add                              Amortized O(1)
  Index                                      O(1)
  Swap Remove                                O(1)
  Clear                                      O(1)
  Ensure Capacity     O(n) only when reallocating

------------------------------------------------------------------------

# Design Choices

## Unmanaged only

`where T : unmanaged`

guarantees:

-   contiguous memory
-   blittable layout
-   no GC tracking
-   pointer access

## Public pointer

The raw pointer is intentionally exposed for engine code and
SIMD-friendly algorithms.

## Swap removal

Wildforge favors dense memory over preserving insertion order.

Most engine systems iterate every element each frame, making cache
locality far more valuable than stable ordering.

## Views

Jobs should consume `UnmanagedView<T>` rather than the owning container.
This prevents accidental structural modification while still allowing
fast read/write access to existing elements.

------------------------------------------------------------------------

# Example

``` csharp
var dinos = new UnmanagedList<Dino>(1024);

dinos.TryAdd(new Dino());

var view = dinos.AsView();

for (int i = 0; i < view.Length; i++)
{
    ref var dino = ref view[i];
    dino.Health--;
}

dinos.Dispose();
```
