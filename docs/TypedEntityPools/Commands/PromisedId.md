# PromisedId

Namespace:

```csharp
using Wildforge.Engine.Core.TypedEntityPools.Commands;
```

`PromisedId` is a temporary identifier representing an entity that **does not exist yet**, but is guaranteed to exist later during command application.

```csharp
public readonly struct PromisedId
{
    public readonly int Value;

    public static readonly PromisedId Invalid = default;

    public bool IsValid => Value > 0;
}
```

Unlike an `EntityHandle`, a `PromisedId` does **not** identify an existing entity.

Instead, it identifies the **future result** of a spawn command.

---

# Why PromisedId Exists

Consider this sequence recorded by different gameplay systems:

```text
Spawn Dinosaur

Spawn Rider

Attach Rider To Dinosaur
```

During simulation, neither entity exists yet.

Therefore:

```text
EntityHandle
```

does not exist.

Yet the attach command must somehow reference both future entities.

That is exactly what `PromisedId` solves.

---

# The Problem

Suppose the rider system wants to record:

```text
Attach Rider

↓

Which Dinosaur?
```

The dinosaur has not been spawned yet.

Its handle cannot exist.

Using:

```csharp
EntityHandle
```

is impossible.

---

# The Solution

Instead:

```text
Spawn Dinosaur

↓

PromisedId = 1
```

```text
Spawn Rider

↓

PromisedId = 2
```

```text
Attach Rider

↓

Dinosaur = 1

Rider = 2
```

Later:

```text
Spawn Commands Apply

↓

PromisedEntityResolver

↓

EntityRefs Created

↓

Attach Command Resolves Both
```

The attach command never needed to know the actual runtime handles.

---

# Typical Workflow

Recording:

```text
Simulation

↓

Spawn Dino (PromisedId 1)

↓

Spawn Rider (PromisedId 2)

↓

Attach Rider (1,2)
```

Application:

```text
Spawn Dino

↓

EntityHandle

↓

EntityRef

↓

Resolver[1]
```

```text
Spawn Rider

↓

EntityHandle

↓

EntityRef

↓

Resolver[2]
```

```text
Attach Rider

↓

Resolve 1

↓

Resolve 2

↓

Perform Attach
```

---

# Example

Recording:

```csharp
var promisedDino =
    allocator.Create();

var promisedRider =
    allocator.Create();

SpawnDinos.TryAdd(
    new SpawnDinoCommand
    {
        PromisedId = promisedDino
    });

SpawnRiders.TryAdd(
    new SpawnRiderCommand
    {
        PromisedId = promisedRider
    });

AttachRiders.TryAdd(
    new AttachRiderCommand
    {
        Dino = promisedDino,
        Rider = promisedRider
    });
```

Nothing exists yet.

Only promises.

---

# During Apply

Spawn command:

```text
PromisedId

↓

Real EntityHandle

↓

EntityRef

↓

Resolver
```

Attach command:

```text
PromisedId

↓

Resolver

↓

EntityRef

↓

Attach
```

---

# Why Not Use EntityHandle?

Entity handles only exist **after** creation.

Timeline:

```text
Simulation

↓

Record Commands

↓

Merge

↓

Apply

↓

Create EntityHandle
```

During recording:

```text
EntityHandle

↓

Does Not Exist
```

Therefore another identifier is required.

---

# Why Not Use Dense Indices?

Dense indices also do not exist.

Even worse:

```text
Dense Index

↓

Unknown

↓

May Change
```

They are completely unsuitable.

---

# Why Not Use GUIDs?

GUIDs identify persistent entities.

Promised IDs identify temporary runtime intentions.

They solve different problems.

GUIDs would be:

- larger
- slower
- unnecessary

A simple integer is sufficient.

---

# Lifetime

A `PromisedId` lives only for one command application.

```text
Record

↓

Merge

↓

Apply

↓

Resolved

↓

Discard
```

It should never survive beyond that.

---

# Resolution

Resolution occurs through:

```text
PromisedEntityResolver
```

Conceptually:

```text
PromisedId

↓

EntityRef
```

Once resolved, the promised ID has completed its purpose.

---

# Multiple Workers

Promised IDs work naturally with parallel recording.

Example:

```text
Worker 0

↓

Spawn Dinos
```

```text
Worker 1

↓

Spawn Riders
```

```text
Worker 2

↓

Attach Riders
```

After merge:

```text
Promised IDs

↓

Resolver

↓

EntityRefs
```

No worker ever needs the runtime handle.

---

# Performance

A `PromisedId` is simply:

```text
int
```

Memory:

```text
4 bytes
```

Generation:

```text
O(1)
```

Resolution:

```text
O(1)
```

Storage:

- unmanaged
- trivially copyable
- cache friendly

---

# Design Decisions

## Temporary Identity

A promised ID is intentionally **not** an entity.

It is only a placeholder.

---

## Runtime Only

Promised IDs never appear in save files.

They disappear immediately after command application.

---

## Integer Based

A simple integer provides:

- fast comparison
- fast indexing
- compact storage
- deterministic generation

Nothing more is required.

---

## Independent Of Entity Pools

Promised IDs exist before any entity pool allocation.

This allows command relationships to be recorded without touching the world.

---

# Best Practices

## Do

Assign a promised ID to every spawn command that may be referenced later.

Resolve promised IDs only during command application.

Discard them after application completes.

---

## Don't

Do not store promised IDs inside gameplay entities.

Do not serialize promised IDs.

Do not confuse promised IDs with entity handles.

Do not resolve promised IDs during simulation.

---

# Relationship To Other Types

```text
PromisedId

↓

PromisedEntityResolver

↓

EntityRef

↓

EntityHandle

↓

EntityPool
```

Promised IDs bridge the gap between **recording commands** and **creating entities**.

---

# Summary

`PromisedId` is Wildforge's solution for referencing entities **before they exist**.

It allows gameplay systems to build complex command graphs—such as spawning entities and immediately establishing relationships—without requiring immediate structural mutation.

Together with `PromisedEntityResolver`, it enables deterministic, parallel-friendly command recording while preserving the integrity of the world's typed entity pools.