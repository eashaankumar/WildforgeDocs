# Lookups and Validation

`EntityPool<T>` provides several ways to access entities.

## IsAlive

```csharp
if (pool.IsAlive(handle))
{
    ...
}
```

Returns true only if:

- the handle ID is valid
- the slot exists
- the slot is alive
- the generation matches

## TryGet

```csharp
if (pool.TryGet(handle, out var ptr))
{
    ptr->Health -= 10;
}
```

This is the standard safe lookup method.

It returns `false` for invalid, stale or destroyed handles.

## UnsafeGet

```csharp
ref var entity = ref pool.UnsafeGet(handle);
```

Use only when the handle is already known to be valid.

Unsafe lookup skips validation.

## Dense Index Lookup

```csharp
pool.TryGetByDenseIndex(index, out var ptr);
```

Dense index lookup is useful for temporary iteration and systems that already operate by dense index.

## Handle From Dense Index

```csharp
pool.TryGetHandleByDenseIndex(index, out var handle);
```

This is useful when a job iterates dense memory but needs to emit a command referencing the current entity.

Example:

```csharp
if (dino.Health <= 0)
{
    pool.TryGetHandleByDenseIndex(i, out var handle);
    commands.DestroyDinos.TryAdd(new DestroyDinoCommand { Handle = handle });
}
```

## Validation Cost

Handle validation is O(1).

It requires:

```text
handle id -> slot
generation compare
alive check
```

For hot systems that process every entity, prefer dense views instead of resolving handles repeatedly.
