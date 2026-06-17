# Parallel Simulation and Command Buffers

`EntityPool<T>` is designed for parallel simulation, but structural mutation must be controlled.

## Simulation Phase

During simulation, workers may:

- read entity fields
- write entity fields in their assigned range
- emit commands

Workers should not:

- create entities directly
- destroy entities directly
- resize pools
- clear pools
- dispose pools

## Structural Phase

After simulation, command buffers are merged and applied in a controlled order.

Example:

```text
SpawnDinos
SpawnRiders
AttachRiders
DestroyProjectiles
DestroyDinos
```

The game defines the correct order.

## Why Commands?

Commands prevent structural mutation while dense views are active.

They also allow worker threads to record intentions without locking the pool.

## Example

During simulation:

```csharp
if (dino.Health <= 0)
{
    commands.DestroyDinos.TryAdd(new DestroyDinoCommand
    {
        Handle = handle
    });
}
```

After simulation:

```csharp
destroyDinos.TryApply(ref world, ref context);
```

## PromisedId

Some commands need to reference entities that do not exist yet.

Example:

```text
Spawn Dino
Spawn Rider
Attach Rider to Dino
```

A `PromisedId` represents a future entity.

During spawn apply:

```text
PromisedId -> EntityRef
```

During attach apply:

```text
resolve PromisedId
write relationship
```

## Worker Command Buffers

The recommended pattern is:

```text
worker 0 commands
worker 1 commands
worker 2 commands

↓

merge

↓

ordered apply
```

This avoids concurrent writes to shared command lists.

## Why This Fits EntityPool

`EntityPool<T>` gives fast dense views for parallel reading/writing.

Command buffers handle structural changes afterward.

Together they preserve:

- high iteration speed
- stable references
- safe structural mutation
- future parallelism
