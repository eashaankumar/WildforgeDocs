# WorldCommandContext

Namespace:

```csharp
using Wildforge.Engine.Core.TypedEntityPools.Commands;
```

`WorldCommandContext` stores temporary state shared by every command while a command buffer is being applied.

```csharp
public struct WorldCommandContext : IDisposable
{
    // Example

    public PromisedEntityResolver PromisedEntities;
}
```

Unlike the game world, the command context is **temporary**.

It exists only during command application.

---

# Purpose

Not every piece of information required by a command belongs inside the world.

Some data exists only while commands are executing.

Examples:

- Promised entity resolution
- Temporary lookup tables
- Statistics
- Debug validation
- Future allocators
- Profiling data

Instead of storing these inside every command, they are collected into a shared context.

---

# Why A Context Exists

Imagine this command:

```csharp
public struct AttachRiderCommand
{
    public PromisedId Rider;

    public PromisedId Dino;
}
```

When this command executes it must resolve both IDs.

Without a shared context every command would need:

```text
PromisedEntityResolver
```

passed individually.

Instead:

```csharp
TryApply(
    ref world,
    ref context);
```

gives every command access to the same temporary services.

---

# World vs Context

The two serve different purposes.

## World

The world owns long-lived game state.

Examples:

```text
Entity Pools

Spatial Indexes

Terrain

Weather

Time

Game State
```

The world survives for the lifetime of the simulation.

---

## Context

The context owns temporary apply-time state.

Examples:

```text
PromisedEntityResolver

Debug Counters

Temporary Allocators

Profiler Data
```

The context exists only while commands execute.

---

# Typical Lifetime

```text
Frame Start

↓

Simulation

↓

Workers Record Commands

↓

Merge

↓

Create Context

↓

Apply Commands

↓

Clear Context

↓

Next Frame
```

The same context object is usually reused every frame.

---

# PromisedEntityResolver

Today the most important member is:

```csharp
PromisedEntityResolver PromisedEntities;
```

Spawn commands register entities:

```text
PromisedId

↓

EntityRef
```

Later commands resolve them.

```text
Attach Rider

↓

Resolve PromisedId

↓

Attach
```

Without the context there would be no shared mapping.

---

# Typical Usage

Creating a context:

```csharp
var context =
    new WorldCommandContext(
        promisedCapacity: 1024);
```

Applying:

```csharp
SpawnDinos.TryApply(
    ref world,
    ref context);

SpawnRiders.TryApply(
    ref world,
    ref context);

AttachRiders.TryApply(
    ref world,
    ref context);
```

Clearing:

```csharp
context.Clear();
```

---

# Why Not Store This In The World?

The resolver is not world state.

It only exists because commands are currently being executed.

Once application finishes:

```text
PromisedId

↓

EntityRef
```

is no longer needed.

Keeping it outside the world avoids unnecessary permanent memory.

---

# Multiple Contexts

Nothing prevents multiple contexts from existing.

For example:

```text
Main Thread Context

Worker Context

Editor Context
```

although the most common usage is one reusable context per command application.

---

# Ownership

The context owns any temporary unmanaged memory it allocates.

Typical lifecycle:

```text
Construct

↓

Reuse

↓

Clear

↓

Dispose
```

Commands never own this memory.

---

# Future Expansion

The context intentionally starts small.

Future engine versions may include:

```text
Temporary Arena

Profiler

Validation State

Error Reporting

Statistics

Scratch Buffers

Debug Flags

Frame Metadata
```

The interface of commands remains unchanged.

Only the context grows.

---

# Thread Safety

A context is intended for one apply operation.

It is **not** designed to be shared between worker threads during recording.

Typical usage:

```text
Workers Record

↓

Merge

↓

Single Thread Apply

↓

Context Used
```

This keeps the implementation simple and avoids synchronization overhead.

---

# Performance

The context itself performs almost no work.

It simply aggregates shared apply-time services.

Commands can access those services directly without repeatedly passing them through every API.

Because the context is reused across frames, it also avoids repeated allocation.

---

# Design Decisions

## Separate World And Temporary State

Long-lived simulation data belongs inside the world.

Temporary apply-time state belongs inside the context.

---

## Shared Services

Rather than every command storing pointers to helper systems, the context provides a single shared access point.

---

## Reusable

The context is designed to be cleared and reused every frame.

This minimizes allocation and improves cache locality.

---

## Extensible

Additional temporary systems can be added later without changing the `IWorldCommand<TWorld>` interface.

Every command automatically gains access to new services through the existing context parameter.

---

# Best Practices

## Do

Reuse one context each frame.

Call:

```csharp
context.Clear();
```

after command application.

Store only temporary services inside the context.

Dispose the context when shutting down the world.

---

## Don't

Do not store permanent game state.

Do not store entity pools.

Do not store world simulation data.

Do not use the context outside command application.

---

# Relationship To Other Types

```text
Simulation

↓

WorldCommandList

↓

WorldCommandContext

↓

IWorldCommand

↓

World
```

The context acts as the bridge between command execution and temporary engine services.

---

# Summary

`WorldCommandContext` is Wildforge's shared apply-time service container.

It provides temporary state required while commands execute, beginning with `PromisedEntityResolver` and expanding to future profiling, validation, and allocator services.

By separating temporary command state from permanent world state, the engine remains cleaner, more reusable, and easier to extend while avoiding unnecessary long-lived memory.