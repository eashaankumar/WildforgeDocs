# IWorldCommand<TWorld>

Namespace:

```csharp
using Wildforge.Engine.Core.TypedEntityPools.Commands;
```

`IWorldCommand<TWorld>` is the fundamental interface used to represent **deferred world mutations**.

```csharp
public interface IWorldCommand<TWorld>
{
    bool TryApply(
        ref TWorld world,
        ref WorldCommandContext context);
}
```

Rather than modifying the world immediately, gameplay systems record commands.

Those commands are executed later during a dedicated **structural phase**.

---

# Why Commands Exist

Wildforge separates simulation into two phases:

```text
Simulation Phase

↓

Structural Phase
```

During simulation:

- AI runs
- physics runs
- animation runs
- gameplay runs

During this phase:

```text
NO structural changes
```

should occur.

Instead, systems record commands.

Later:

```text
Worker Buffers

↓

Merge

↓

Sequential Apply

↓

World Updated
```

This avoids invalidating entity views while jobs are still executing.

---

# The Problem

Suppose AI does this:

```csharp
if (dino.Health <= 0)
{
    world.Dinos.TryDestroy(handle);
}
```

Meanwhile another worker is still iterating:

```csharp
var view = world.Dinos.AsView();
```

Destroying an entity causes:

- swap-remove
- dense index movement
- view invalidation

This is unsafe.

---

# The Solution

Instead:

```csharp
if (dino.Health <= 0)
{
    commands.DestroyDinos.TryAdd(
        new DestroyDinoCommand
        {
            Handle = handle
        });
}
```

Nothing changes immediately.

The world remains stable.

Later:

```csharp
DestroyDinos.TryApply(
    ref world,
    ref context);
```

The mutation occurs after simulation finishes.

---

# Interface

```csharp
public interface IWorldCommand<TWorld>
{
    bool TryApply(
        ref TWorld world,
        ref WorldCommandContext context);
}
```

Every command implements exactly one operation.

Examples:

```text
Spawn Dino

Destroy Projectile

Attach Rider

Spawn Plant

Create Item

Remove Tree
```

---

# Why TryApply?

Wildforge follows the engine-wide convention:

```text
TryX

UnsafeX
```

Instead of throwing exceptions for expected failures.

Examples of legitimate failures:

```text
Entity already destroyed

Pool full

Invalid handle

PromisedId not found

Duplicate registration
```

Returning:

```text
false
```

allows the caller to decide how to respond.

---

# World Parameter

```csharp
ref TWorld world
```

The command receives the world by reference.

The world owns:

- entity pools
- spatial indexes
- navigation
- game state

The command performs exactly one mutation.

Example:

```csharp
public bool TryApply(
    ref ForgewildWorld world,
    ref WorldCommandContext context)
{
    return world.Projectiles.TryDestroy(Handle);
}
```

---

# WorldCommandContext

Commands also receive:

```csharp
ref WorldCommandContext
```

The context stores temporary state required during application.

Examples:

- promised entity resolver
- temporary allocators
- statistics
- validation
- future debugging data

Keeping this separate prevents commands from storing unnecessary state.

---

# Example

Destroy command:

```csharp
public struct DestroyDinoCommand
    : IWorldCommand<ForgewildWorld>
{
    public EntityHandle Handle;

    public bool TryApply(
        ref ForgewildWorld world,
        ref WorldCommandContext context)
    {
        return world.Dinos.TryDestroy(
            Handle);
    }
}
```

---

Spawn command:

```csharp
public struct SpawnPlantCommand
    : IWorldCommand<ForgewildWorld>
{
    public Plant Plant;

    public bool TryApply(
        ref ForgewildWorld world,
        ref WorldCommandContext context)
    {
        return world.Plants.TryCreate(
            Plant,
            out _);
    }
}
```

---

# Promised Entities

Commands can cooperate.

Suppose:

```text
Spawn Dino

↓

Spawn Rider

↓

Attach Rider
```

The rider cannot know the dino's runtime handle yet.

Instead:

```text
PromisedId
```

is recorded.

During application:

```text
Spawn Dino

↓

PromisedId

↓

EntityRef
```

Later:

```text
Attach Rider

↓

Resolve PromisedId

↓

Attach
```

This allows complex creation chains without immediate structural mutation.

---

# Thread Safety

Commands themselves contain no synchronization.

Instead:

```text
Worker 0

↓

WorkerCommandBuffer
```

```text
Worker 1

↓

WorkerCommandBuffer
```

```text
Worker 2

↓

WorkerCommandBuffer
```

Workers never write into the same command list.

Later:

```text
Merge

↓

Main Buffer

↓

Sequential Apply
```

This completely removes locking from the hot simulation phase.

---

# Command Lifetime

Commands are temporary.

Typical lifetime:

```text
Record

↓

Merge

↓

Apply

↓

Discard
```

Commands are **not** persistent objects.

---

# Design Philosophy

Each command should perform one logical operation.

Good:

```text
Destroy Projectile

Spawn Dino

Attach Rider

Set Target

Remove Plant
```

Poor:

```text
Update Entire World
```

Small commands compose well.

Large commands become difficult to reason about.

---

# Typed Commands

Wildforge intentionally avoids:

```text
Base Command

↓

virtual Apply()
```

Instead:

```text
WorldCommandList<
    TWorld,
    TCommand>
```

stores one command type.

Advantages:

- no boxing
- no virtual dispatch
- contiguous memory
- cache friendly
- compile-time validation

---

# Unmanaged Commands

Commands should generally be:

```csharp
struct
```

and:

```text
unmanaged
```

This allows them to be stored inside:

- `UnmanagedList<T>`
- `WorldCommandList<TWorld,TCommand>`
- worker-local buffers

without creating managed allocations.

---

# Performance

Applying a command is:

```text
O(1)
```

for most commands.

Examples:

```text
Destroy Entity

Spawn Entity

Resolve PromisedId

Attach Relationship
```

The dominant cost is usually the underlying world operation rather than the command dispatch itself.

---

# Best Practices

## Do

Keep commands small.

Use unmanaged structs.

Store handles instead of pointers.

Record during simulation.

Apply afterward.

Return `false` instead of throwing expected runtime failures.

---

## Don't

Do not mutate the world directly inside gameplay systems.

Do not store managed references.

Do not create one giant generic command type.

Do not use inheritance for command polymorphism.

Do not perform unrelated work inside one command.

---

# Relationship To Other Types

```text
Gameplay System

↓

Record Command

↓

WorldCommandList

↓

Merge

↓

TryApply()

↓

World
```

Commands work together with:

- `WorldCommandList<TWorld,TCommand>`
- `WorldCommandBuffer`
- `WorldCommandContext`
- `PromisedId`
- `PromisedEntityResolver`

to provide safe structural mutation for parallel simulation.

---

# Summary

`IWorldCommand<TWorld>` is the foundation of Wildforge's deferred mutation system.

It allows gameplay systems to describe **what should happen** without immediately modifying the world.

This enables:

- stable entity iteration
- safe parallel simulation
- worker-local command recording
- deterministic structural updates
- high-performance typed command buffers

while keeping world mutation explicit, predictable, and easy to reason about.