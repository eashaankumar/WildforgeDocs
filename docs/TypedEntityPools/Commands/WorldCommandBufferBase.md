# WorldCommandList<TWorld, TCommand>

Namespace:

```csharp
using Wildforge.Engine.Core.TypedEntityPools.Commands;
```

`WorldCommandList<TWorld, TCommand>` is Wildforge's high-performance container for recording and applying a single type of world command.

```csharp
public unsafe struct WorldCommandList<TWorld, TCommand>
    where TCommand : unmanaged, IWorldCommand<TWorld>
```

Unlike traditional command systems that store heterogeneous command objects in a single collection, Wildforge stores **one command type per list**.

This matches the engine's overall philosophy:

- Typed pools
- Typed systems
- Typed commands
- Dense unmanaged memory

---

# Philosophy

Commands should be stored exactly like entities:

```text
Dense

Contiguous

Cache Friendly

Unmanaged
```

Instead of:

```text
Command

↓

virtual Apply()

↓

Heap Allocation

↓

Virtual Dispatch
```

Wildforge stores:

```text
SpawnDinoCommand[]

DestroyDinoCommand[]

AttachRiderCommand[]
```

Each command type gets its own contiguous array.

---

# Why One List Per Command?

Suppose your frame records:

```text
Spawn Dino

Destroy Projectile

Spawn Plant

Spawn Dino

Attach Rider

Destroy Dino
```

A traditional command buffer becomes:

```text
Mixed Objects

↓

Dynamic Casts

↓

Virtual Calls
```

Wildforge instead stores:

```text
SpawnDinos

SpawnPlant

DestroyProjectiles

DestroyDinos

AttachRiders
```

Each list is homogeneous.

This allows:

- contiguous memory
- zero boxing
- zero virtual dispatch
- simple iteration

---

# Structure

Internally, a command list is simply an unmanaged list of commands.

Conceptually:

```text
SpawnDinoCommand

↓

UnmanagedList<SpawnDinoCommand>
```

The container owns:

- recorded commands
- capacity
- unmanaged memory

---

# Recording Commands

Commands are recorded during simulation.

Example:

```csharp
DestroyDinos.TryAdd(
    new DestroyDinoCommand
    {
        Handle = handle
    });
```

Nothing happens immediately.

The command simply becomes part of the list.

---

# Applying Commands

Later:

```csharp
DestroyDinos.TryApply(
    ref world,
    ref context);
```

The list executes:

```text
Command 0

↓

Command 1

↓

Command 2

↓

...
```

in insertion order.

---

# Execution Order

Commands inside a list always execute in the order they were recorded.

Example:

```text
Spawn A

Spawn B

Spawn C
```

Application order:

```text
Spawn A

↓

Spawn B

↓

Spawn C
```

This deterministic ordering makes debugging significantly easier.

---

# Why Separate Lists?

Imagine:

```text
Spawn Dino

Attach Rider

Spawn Rider
```

This order is incorrect.

Instead:

```text
Spawn Dinos

↓

Spawn Riders

↓

Attach Riders
```

The game controls the order by deciding **which command lists execute first**.

This is impossible with one giant heterogeneous command list.

---

# Worker Recording

Every worker should own its own command buffer.

```text
Worker 0

↓

SpawnDinos
```

```text
Worker 1

↓

SpawnDinos
```

```text
Worker 2

↓

SpawnDinos
```

Workers never contend for the same memory.

No locks are required.

---

# Merging

After simulation:

```text
Worker 0

↓

Worker 1

↓

Worker 2
```

becomes:

```text
Main SpawnDino List
```

Conceptually:

```csharp
main.TryAppendFrom(worker0);

main.TryAppendFrom(worker1);

main.TryAppendFrom(worker2);
```

The merge phase is intentionally separate from simulation.

---

# Applying After Merge

Once merged:

```text
Spawn Dinos

↓

Spawn Riders

↓

Attach Riders

↓

Destroy Projectiles
```

The game chooses the correct ordering.

This keeps structural mutation deterministic.

---

# Failure Handling

Every command returns:

```text
true

or

false
```

Typical failures include:

- invalid handle
- pool full
- duplicate registration
- missing promised ID

The command list immediately reports failure to the caller.

How the game responds is game-specific.

Possible strategies include:

- abort remaining commands
- continue
- log warning
- retry later

---

# Capacity

Like every unmanaged container:

```csharp
TryEnsureCapacity()

TryTrimExcess()

Dispose()
```

Large simulations should preallocate.

Example:

```csharp
SpawnDinos.TryEnsureCapacity(
    10000);
```

to avoid reallocations during gameplay.

---

# Clearing

After successful application:

```csharp
SpawnDinos.Clear();
```

The allocated memory remains.

Only the logical count becomes zero.

This makes the next frame allocation-free.

---

# Lifetime

Typical lifetime:

```text
Begin Frame

↓

Record Commands

↓

Merge

↓

Apply

↓

Clear

↓

Next Frame
```

Command lists are reused every frame.

They should **not** be recreated every frame.

---

# Command Size

Commands should generally remain small.

Good:

```text
EntityHandle

float

int

EntityRef
```

Avoid storing enormous gameplay objects.

Instead of:

```text
SpawnDinoCommand

↓

Entire 4 KB Dino
```

prefer:

```text
Spawn Parameters

↓

Construct
```

or

```text
Pointer Fill
```

during application.

---

# Performance

Recording:

```text
O(1)
```

Appending worker buffers:

```text
O(n)
```

where:

```text
n
```

is the number of commands being merged.

Applying:

```text
O(n)
```

with each command usually performing:

```text
O(1)
```

world mutations.

The command list itself introduces almost no overhead.

---

# Relationship To WorldCommandBuffer

A command list stores **one command type**.

A command buffer owns **many command lists**.

Example:

```text
ForgewildCommandBuffer

├── SpawnDinos
├── SpawnPlants
├── SpawnProjectiles
├── DestroyDinos
├── DestroyPlants
├── DestroyProjectiles
├── AttachRiders
├── SetTargets
└── ...
```

The buffer coordinates ordering.

The lists simply store commands.

---

# Best Practices

## Do

Use one command type per list.

Preallocate expected capacity.

Reuse lists every frame.

Merge worker-local lists after simulation.

Apply in deterministic order.

Store unmanaged command structs.

---

## Don't

Do not mix unrelated commands together.

Do not allocate a new list every frame.

Do not execute structural mutations while recording.

Do not share one command list across worker threads.

Do not rely on recording order across different command types.

---

# Design Decisions

## Typed Everything

Wildforge intentionally avoids generic runtime command polymorphism.

Compile-time typing provides:

- faster code
- better IDE support
- easier debugging
- fewer runtime surprises

---

## Dense Memory

Commands are stored contiguously.

This improves cache locality during both merge and application.

---

## Zero GC

Because commands are unmanaged structs stored inside unmanaged containers:

- no boxing
- no heap allocations
- no garbage collection pressure

during normal gameplay.

---

## Engine Agnostic

The engine does not know what a command means.

It only knows:

```text
Record

↓

Merge

↓

Apply
```

The game defines:

- command types
- ordering
- world behavior

---

# Summary

`WorldCommandList<TWorld, TCommand>` is Wildforge's high-performance typed command container.

It provides:

- dense unmanaged storage
- zero-allocation recording
- deterministic execution
- worker-local recording
- efficient merging
- cache-friendly application

and serves as the fundamental building block for Wildforge's deferred structural mutation system.