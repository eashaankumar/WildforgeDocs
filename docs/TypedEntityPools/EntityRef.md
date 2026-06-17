# EntityRef

Namespace:

```csharp
using Wildforge.Engine.Core.TypedEntityPools.Entities;
```

`EntityRef` is Wildforge's universal runtime reference for entities across **all typed entity pools**.

```csharp
public readonly struct EntityRef
{
    public readonly EntityKind Kind;
    public readonly EntityHandle Handle;

    public static readonly EntityRef Invalid = default;

    public bool IsValid =>
        Kind.IsValid &&
        Handle.IsValid;
}
```

While an `EntityHandle` identifies an entity **within one specific pool**, an `EntityRef` identifies an entity **within the entire world**.

It is the standard way for entities of different types to reference one another.

---

# Why EntityRef Exists

Suppose your world contains:

```text
World

├── DinoPool
├── MammalPool
├── PlantPool
├── ProjectilePool
├── ItemPool
├── PlayerPool
```

A dinosaur might target:

- another dinosaur
- a mammal
- a player
- a plant

A projectile might belong to:

- a player
- a turret
- a dinosaur

A plant might have been planted by:

- a player
- an NPC

None of these relationships can be represented by an `EntityHandle` alone.

Instead they use:

```text
EntityKind

+

EntityHandle

↓

EntityRef
```

---

# Structure

```csharp
public readonly struct EntityRef
{
    public readonly EntityKind Kind;

    public readonly EntityHandle Handle;
}
```

The struct intentionally contains only two values.

```text
EntityKind

↓

Which pool?
```

```text
EntityHandle

↓

Which entity?
```

Together they uniquely identify one runtime entity.

---

# EntityKind

The first field tells the world which pool owns the entity.

Example:

```text
Kind

↓

Dino
```

or

```text
Projectile
```

or

```text
Plant
```

The engine does not assign meaning to kinds.

Games define their own catalogs.

---

# EntityHandle

The second field identifies one entity inside that pool.

Because handles use generations, stale references remain invalid even after slot reuse.

---

# Validity

A reference is structurally valid when:

```text
Kind.IsValid

AND

Handle.IsValid
```

This does **not** guarantee the entity still exists.

The world must resolve the reference.

---

# Resolving References

Typically, the game world owns every entity pool.

Resolving a reference looks something like:

```csharp
switch(reference.Kind.Value)
{
    case ForgewildEntityKinds.Dino:
        return Dinos.TryGet(
            reference.Handle,
            out var dino);

    case ForgewildEntityKinds.Plant:
        return Plants.TryGet(
            reference.Handle,
            out var plant);

    case ForgewildEntityKinds.Projectile:
        return Projectiles.TryGet(
            reference.Handle,
            out var projectile);
}
```

The switch belongs inside the game.

Not the engine.

---

# Why Not One Giant Entity Pool?

Many engines store every entity together.

```text
World

↓

EntityPool<Entity>
```

Wildforge intentionally avoids this.

Instead:

```text
World

↓

Typed Pools
```

Benefits include:

- better cache locality
- compile-time typing
- no component casting
- simpler gameplay code
- easier debugging

`EntityRef` provides cross-pool references without sacrificing typed storage.

---

# Typical Usage

## AI Targets

```csharp
public struct Dino
{
    public EntityRef Target;
}
```

The target might be:

- another dino
- mammal
- player
- plant

The AI doesn't care.

---

## Projectile Owner

```csharp
public struct Projectile
{
    public EntityRef Owner;
}
```

Ownership survives regardless of the owner's type.

---

## Parent Relationships

```csharp
public struct Rider
{
    public EntityRef MountedEntity;
}
```

A rider could mount:

- dinosaur
- mammoth
- horse

No separate fields are required.

---

## Gameplay Interactions

```csharp
public struct DamageEvent
{
    public EntityRef Attacker;

    public EntityRef Victim;
}
```

The damage system can remain completely generic.

---

# Runtime Lifetime

An `EntityRef` is a runtime reference.

It becomes invalid if:

- the entity is destroyed
- the handle becomes stale
- the world is destroyed

Do not store entity references permanently.

---

# Runtime vs Persistence

Like `EntityHandle`:

`EntityRef` should **not** be serialized.

Instead:

```text
Persistent ID

↓

Load World

↓

Resolve

↓

EntityRef
```

Persistent identity belongs in a dedicated registry.

---

# EntityRef vs EntityHandle

Use `EntityHandle` when you already know the pool.

Example:

```csharp
world.Dinos
```

Inside that pool:

```csharp
EntityHandle Parent;
```

is sufficient.

---

Use `EntityRef` when the target might belong to multiple pools.

Example:

```csharp
public EntityRef Target;
```

---

# EntityRef vs Pointer

Pointers are temporary.

Entity references are stable.

Pointers should only exist inside:

- jobs
- hot loops
- frame-local algorithms

Gameplay relationships should always use:

```text
EntityRef
```

---

# EntityRef vs Persistent IDs

Persistent IDs answer:

```text
Which entity across saves?
```

Entity references answer:

```text
Which runtime entity right now?
```

These are different problems.

---

# Performance

`EntityRef` contains:

```text
EntityKind

+

EntityHandle
```

Memory:

```text
ushort
padding
int
int
```

Approximately:

```text
12–16 bytes
```

depending on alignment.

Resolution is:

```text
O(1)
```

consisting of:

- world pool lookup
- handle validation
- slot lookup

---

# Design Decisions

## Fully Typed World

The world owns typed pools.

The reference simply selects one.

No dynamic casting.

No runtime RTTI.

---

## Small Value Type

`EntityRef` is intentionally lightweight.

It can safely be stored inside:

- entities
- command buffers
- AI state
- events
- unmanaged containers

---

## No Pool Pointer

The reference does not store pointers.

Instead:

```text
Kind

↓

World

↓

Pool

↓

Handle
```

This keeps references:

- deterministic
- unmanaged
- serializable
- portable

---

## No Reflection

Kinds are numeric.

No runtime reflection is required.

---

# Best Practices

## Do

Use `EntityRef` whenever the target can belong to different entity pools.

Store references inside unmanaged entity structs.

Resolve through the world.

---

## Don't

Do not store raw pointers.

Do not store dense indices.

Do not serialize runtime references.

Do not manually resolve kinds outside the world layer.

---

# Relationship To Other Types

```text
EntityKind

↓

Identifies a pool
```

```text
EntityHandle

↓

Identifies an entity
inside that pool
```

```text
EntityRef

↓

Combines both
```

```text
World

↓

Resolves EntityRef

↓

Returns entity
```

---

# Common Workflow

```text
Projectile

↓

Owner

↓

EntityRef

↓

World.Resolve()

↓

Player
```

or

```text
Projectile

↓

Owner

↓

EntityRef

↓

World.Resolve()

↓

Dinosaur
```

The gameplay code doesn't need to know which.

---

# Summary

`EntityRef` is Wildforge's universal runtime entity reference.

It combines:

- `EntityKind`
- `EntityHandle`

to identify any entity stored in any typed pool.

It enables:

- cross-pool relationships
- strongly typed entity storage
- compact unmanaged data
- O(1) resolution
- stable runtime references

while preserving the performance advantages of Wildforge's typed dense entity architecture.