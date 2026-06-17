# Patterns, Best Practices and Pitfalls

## Prefer Views for Systems

Use:

```csharp
var view = pool.AsView();
```

for systems that update all entities.

Avoid repeated handle lookup in full-pool loops.

## Use Handles for Identity

Use `EntityHandle` for references that survive beyond one loop iteration.

Do not store dense indices.

Dense indices can change after destruction.

## Use EntityRef Across Pools

Use `EntityRef` when the target can belong to different typed pools.

```csharp
public EntityRef Target;
```

## Do Not Mutate Structure During Iteration

Bad:

```csharp
for (...)
{
    pool.TryDestroy(handle);
}
```

Good:

```csharp
commands.DestroyDinos.TryAdd(...);
```

Then apply after iteration.

## Use In-Place Creation for Large Structs

Large entity structs should be filled directly in pool memory.

```csharp
pool.TryCreate(out handle, out ptr);
Fill(ptr);
```

## Preallocate

Use known world budgets to preallocate.

```csharp
new EntityPool<Dino>(maxDinos);
```

or reserve later.

## Do Not Rely on Order

Destroy uses swap-remove.

Iteration order is unstable.

If order matters, maintain a separate ordering structure.

## Dispose Owners

`EntityPool<T>` owns unmanaged memory.

Always call `Dispose()` when done.

## Avoid Managed References in T

`T` must be unmanaged.

Do not store managed objects, strings, arrays or class references inside entity structs.

Use IDs, handles, fixed arrays, unmanaged views, or external registries instead.

## Common Mistakes

### Storing Dense Index

Dense indices are temporary.

They are not identity.

### Destroying While Iterating

This can invalidate views and change dense order.

### Sharing Command Lists Across Workers

Use worker-local command buffers and merge later.

### Using Full Pool in Jobs

Jobs should usually receive views, not pools.
