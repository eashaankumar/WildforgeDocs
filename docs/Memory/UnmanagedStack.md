# UnmanagedStack`<T>`{=html}

Namespace:

``` csharp
using Wildforge.Engine.Core.Memory;
```

`UnmanagedStack<T>` is Wildforge's unmanaged last-in, first-out (LIFO)
container.

``` csharp
public unsafe struct UnmanagedStack<T>
    where T : unmanaged
```

Internally, it is built on top of `UnmanagedList<T>` and provides a
specialized API for stack semantics while retaining contiguous unmanaged
storage.

------------------------------------------------------------------------

# Purpose

Use `UnmanagedStack<T>` whenever elements should be processed in reverse
order of insertion.

Typical uses include:

-   free-slot lists
-   depth-first traversal
-   temporary work stacks
-   parser state
-   undo stacks
-   custom allocators
-   graph traversal
-   engine scratch data

Wildforge currently uses `UnmanagedStack<T>` internally for reusable
slot management inside `EntityPool<T>`.

------------------------------------------------------------------------

# Design Philosophy

Unlike `Stack<T>` from the .NET runtime, `UnmanagedStack<T>`:

-   stores elements in unmanaged memory
-   never allocates managed objects during normal operation
-   exposes raw pointers
-   supports direct iteration through `UnmanagedView<T>`
-   integrates naturally with the rest of Wildforge's unmanaged
    containers

Because the stack is backed by a contiguous array, iteration is
cache-friendly and extremely fast.

------------------------------------------------------------------------

# Construction

``` csharp
var stack = new UnmanagedStack<int>(256);
```

The constructor reserves space for the requested capacity.

The stack grows automatically when additional space is required.

------------------------------------------------------------------------

# Pushing

Safe:

``` csharp
stack.TryPush(value);
```

Unsafe:

``` csharp
stack.UnsafePush(value);
```

`TryPush` performs any required capacity management.

`UnsafePush` assumes sufficient capacity already exists.

------------------------------------------------------------------------

# Popping

Safe:

``` csharp
if (stack.TryPop(out var value))
{
    // use value
}
```

Unsafe:

``` csharp
var value = stack.UnsafePop();
```

`UnsafePop` should only be used when the caller knows the stack is not
empty.

------------------------------------------------------------------------

# Inspecting Memory

The stack exposes its underlying pointer.

``` csharp
int* ptr = stack.Ptr;
```

The active element count is available through:

``` csharp
int count = stack.Count;
```

The current reserved capacity is:

``` csharp
int capacity = stack.Capacity;
```

------------------------------------------------------------------------

# Views

Like every contiguous Wildforge container, the stack can expose a
lightweight view.

``` csharp
UnmanagedView<int> view = stack.AsView();
```

Views provide:

-   pointer
-   length

without exposing ownership or structural operations.

Views are intended for:

-   algorithms
-   jobs
-   serialization
-   debugging
-   bulk processing

------------------------------------------------------------------------

# Capacity Management

Reserve memory:

``` csharp
stack.TryEnsureCapacity(1024);
```

Resize explicitly:

``` csharp
stack.TrySetCapacity(2048);
```

Shrink unused memory:

``` csharp
stack.TryTrimExcess();
```

------------------------------------------------------------------------

# Lifetime

Remove all elements while retaining memory:

``` csharp
stack.Clear();
```

Release unmanaged memory:

``` csharp
stack.Dispose();
```

After disposal, the stack should not be accessed again.

------------------------------------------------------------------------

# Performance

  Operation                              Complexity
  ----------------- -------------------------------
  Push                               Amortized O(1)
  Pop                                          O(1)
  Peek via Ptr                                 O(1)
  Clear                                        O(1)
  Ensure Capacity     O(n) only during reallocation

------------------------------------------------------------------------

# Design Choices

## Built on UnmanagedList

The implementation intentionally reuses `UnmanagedList<T>` rather than
maintaining a separate allocator.

This keeps behavior consistent across containers while minimizing
implementation complexity.

## Contiguous Memory

Elements remain tightly packed in memory.

This improves cache locality during iteration and bulk processing.

## Pointer Access

The underlying pointer is exposed because many engine systems operate
directly on unmanaged memory.

## Views

Jobs and algorithms should consume `UnmanagedView<T>` whenever ownership
is unnecessary.

This avoids accidental structural mutation while still providing direct
access to active elements.

------------------------------------------------------------------------

# Example

``` csharp
var freeSlots = new UnmanagedStack<int>(256);

freeSlots.TryPush(1);
freeSlots.TryPush(2);
freeSlots.TryPush(3);

while (freeSlots.TryPop(out var slot))
{
    Console.WriteLine(slot);
}

freeSlots.Dispose();
```

------------------------------------------------------------------------

# Relationship to UnmanagedList

`UnmanagedStack<T>` is a specialized wrapper around `UnmanagedList<T>`.

Choose:

-   `UnmanagedList<T>` when arbitrary indexing or swap-removal is
    required.
-   `UnmanagedStack<T>` when last-in, first-out behavior is desired.

Both containers share the same unmanaged memory model, capacity
management strategy, and view system, making them consistent to use
throughout the engine.
