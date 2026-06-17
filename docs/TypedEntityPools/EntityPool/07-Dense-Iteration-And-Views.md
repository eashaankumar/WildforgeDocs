# Dense Iteration and Views

Dense iteration is the primary way systems should process entities.

```csharp
var view = pool.AsView();

for (int i = 0; i < view.Count; i++)
{
    ref var entity = ref view[i];
}
```

## EntityPoolView<T>

A view contains:

```csharp
public readonly T* Ptr;
public readonly int Count;
public readonly uint StructuralVersion;
```

It is a lightweight snapshot of the pool's current dense storage pointer and count.

## Why Views Exist

Views allow jobs and systems to process entity memory without exposing the owning pool.

A view can read and write existing entities, but it cannot:

- create entities
- destroy entities
- resize memory
- clear the pool
- dispose storage

This separation is important for parallel simulation.

## Structural Version

The view stores the pool's structural version.

Structural changes increment the version.

Examples:

- create
- destroy
- clear
- dispose
- capacity changes

The version can be used by debug validation to detect invalidated views.

## Structural Rule

Do not structurally modify a pool while a view is being used.

Allowed:

```text
read existing fields
write existing fields
```

Not allowed:

```text
create entity
destroy entity
resize storage
clear pool
dispose pool
```

## Jobs

Jobs should receive views, not owning pools.

```csharp
public unsafe struct DinoUpdateJob : IWildJob
{
    public EntityPool<Dino>.EntityPoolView<Dino> Dinos;

    public void Execute(int start, int count, int workerIndex)
    {
        for (int i = start; i < start + count; i++)
        {
            ref var dino = ref Dinos[i];
            Update(ref dino);
        }
    }
}
```

## Why Dense Iteration Is Fast

Dense iteration walks memory linearly.

This allows hardware prefetching and minimizes cache misses.

The CPU sees:

```text
Dino
Dino
Dino
Dino
Dino
```

instead of chasing scattered references.

This is the primary performance advantage of the architecture.
