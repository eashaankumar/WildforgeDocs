# Performance Characteristics

`EntityPool<T>` is designed to make container overhead negligible.

## Complexity

| Operation | Complexity |
|-----------|-----------:|
| Create | Amortized O(1) |
| Destroy | O(1) |
| IsAlive | O(1) |
| TryGet | O(1) |
| Dense iteration | O(n) |
| Clear | O(n) over slots |
| Capacity growth | O(n) only when reallocating |

## Zero Managed Allocations

Entity data is stored in unmanaged buffers.

Normal pool operations do not allocate managed memory.

This avoids:

- garbage collection pressure
- GC spikes
- hidden object churn
- managed array relocation

## Benchmark Findings

Large gameplay entity benchmarks showed that operations such as:

- bulk spawn
- massive read
- massive write
- AI update
- physics update
- gameplay update

cluster closely in runtime.

This indicates the pool is primarily memory-bandwidth limited rather than algorithm limited.

That is the desired result.

## Pointer Creation

For large structs, in-place creation is faster than value creation.

```csharp
pool.TryCreate(out handle, out ptr);
```

This avoids copying a large temporary struct into pool memory.

## Fragmentation Resistance

Because destroyed dense slots are filled with the final entity, dense storage remains compact.

Free slots are reused for stable IDs.

The pool avoids sparse holes in the hot iteration array.

## Preallocation

For best performance:

```csharp
pool.TryEnsureCapacity(expectedLiveCount);
pool.TryEnsureSlotCapacity(expectedMaxSlotCount);
```

This reduces allocation and copying during gameplay.

## Main Bottleneck

For large entities, the bottleneck becomes physical memory bandwidth.

That means the pool is no longer the limiting abstraction.
