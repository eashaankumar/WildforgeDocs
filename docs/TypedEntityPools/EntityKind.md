# EntityKind

Namespace:

```csharp
using Wildforge.Engine.Core.TypedEntityPools.Entities;
```

`EntityKind` identifies which typed entity pool an entity belongs to.

```csharp
public readonly struct EntityKind
{
    public readonly ushort Value;

    public static readonly EntityKind None = default;

    public bool IsValid => Value != 0;
}
```

`EntityKind` exists because an `EntityHandle` only identifies an entity **within one specific pool**.

A handle alone answers:

> Which entity?

An `EntityKind` answers:

> Which pool?

Together they form an `EntityRef`.

---

# Why EntityKind Exists

Suppose your game has:

```text
World

├── Dinos
├── Mammals
├── Plants
├── Projectiles
├── Items
├── Players
```

Now consider this handle:

```text
Id = 25
Generation = 3
```

Which entity is it?

It could be:

- Dino #25
- Plant #25
- Projectile #25
- Item #25

The handle has no way of knowing.

Instead we pair it with an `EntityKind`.

```text
Kind = Dino

Handle

↓

Actual Dino
```

---

# Why Not Store A Pool Pointer?

One option would be:

```text
EntityHandle

↓

Pool Pointer

↓

Slot
```

Wildforge intentionally avoids this.

Reasons:

- larger handles
- more memory traffic
- difficult serialization
- harder testing
- tighter coupling
- more pointer chasing

Instead:

```text
EntityKind

↓

Game World

↓

Correct Pool

↓

Handle
```

The world already owns every pool.

---

# Engine Philosophy

The engine deliberately **does not know** what a dinosaur is.

Or a mammal.

Or a player.

Or a projectile.

Those are **game concepts**.

The engine only knows:

```text
EntityKind
```

which is simply an identifier.

Games decide what every kind means.

---

# Why Not An Enum?

Many engines expose:

```csharp
enum EntityKind
{
    Dino,
    Plant,
    Projectile
}
```

Wildforge intentionally avoids this.

Why?

Because the engine should not know anything about game-specific entity types.

Instead the engine provides a tiny value type:

```csharp
public readonly struct EntityKind
{
    public readonly ushort Value;
}
```

The game defines the catalog.

---

# Defining Game Entity Kinds

Forgewild might define:

```csharp
public static class ForgewildEntityKinds
{
    public static readonly EntityKind Dino = new(1);

    public static readonly EntityKind Mammal = new(2);

    public static readonly EntityKind Plant = new(3);

    public static readonly EntityKind Projectile = new(4);

    public static readonly EntityKind Item = new(5);

    public static readonly EntityKind Player = new(6);
}
```

Another game might define something completely different.

```text
Spaceship
Planet
Missile
Station
Pilot
```

The engine code remains identical.

---

# Why ushort?

Wildforge uses:

```csharp
ushort
```

instead of:

```text
byte
```

because games often grow far beyond a few hundred entity types.

A `ushort` allows:

```text
65,536
```

possible entity kinds while remaining compact.

Memory usage:

```text
ushort

2 bytes
```

which is negligible compared to the entity data itself.

---

# Invalid Kind

Value zero is reserved.

```csharp
EntityKind.None
```

or

```text
Value = 0
```

represents:

```text
No entity type.
```

Therefore:

```csharp
default(EntityKind)
```

is automatically invalid.

This mirrors the design of `EntityHandle`.

---

# Typical Usage

Most gameplay code never manipulates `EntityKind` directly.

Instead it appears inside:

```csharp
EntityRef
```

For example:

```csharp
public struct Projectile
{
    public EntityRef Owner;
}
```

The owner might be:

- Dino
- Mammal
- Player
- Turret

The projectile doesn't care.

---

# Resolving EntityKind

A world resolves kinds.

For example:

```csharp
switch(entity.Kind.Value)
{
    case 1:
        return World.Dinos.TryGet(
            entity.Handle,
            out var dino);

    case 2:
        return World.Plants.TryGet(
            entity.Handle,
            out var plant);

    case 3:
        return World.Projectiles.TryGet(
            entity.Handle,
            out var projectile);
}
```

The switch belongs to the game.

Not the engine.

---

# EntityKind + EntityHandle

Neither is sufficient by itself.

```text
EntityHandle

↓

Which entity?

```

```text
EntityKind

↓

Which pool?
```

Together:

```text
EntityRef

↓

Exactly one runtime entity.
```

---

# Runtime Only

Like `EntityHandle`, `EntityKind` is a runtime concept.

It should not be treated as a persistent asset identifier.

Instead:

```text
Save File

↓

Persistent IDs

↓

World Load

↓

EntityKind
+
EntityHandle
```

---

# Performance

An `EntityKind` is:

```text
2 bytes
```

Comparisons are:

```text
O(1)
```

The value is trivially copyable.

It may be safely embedded inside:

- entities
- command buffers
- unmanaged containers
- fixed arrays

---

# Design Decisions

## Game-Owned Catalog

The engine owns the type.

The game owns the meaning.

This keeps Wildforge reusable across completely different genres.

---

## No Reflection

Entity kinds are simple numeric identifiers.

No reflection.

No runtime type lookup.

No string comparisons.

---

## No Registration System

Wildforge intentionally avoids automatic registration.

Games should explicitly define their entity kinds.

This makes entity identities deterministic and easy to debug.

---

## Unmanaged

`EntityKind` is fully unmanaged.

It can safely be stored anywhere an unmanaged type is required.

---

# Best Practices

## Do

Create one central catalog.

```csharp
ForgewildEntityKinds
```

Reserve:

```text
0
```

as invalid.

Use `EntityKind` together with `EntityHandle`.

---

## Don't

Do not expose engine enums containing game types.

Do not perform string lookups during gameplay.

Do not use reflection to determine kinds.

Do not treat `EntityKind` as a persistent asset identifier.

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

Identifies one entity
inside that pool
```

```text
EntityRef

↓

EntityKind
+
EntityHandle
```

```text
World

↓

Uses EntityKind

↓

Finds correct EntityPool<T>

↓

Uses EntityHandle

↓

Returns entity
```

---

# Summary

`EntityKind` is Wildforge's engine-level abstraction for identifying **which typed entity pool** a runtime entity belongs to.

By separating pool identity from entity identity, Wildforge allows:

- fully typed entity pools
- game-defined entity catalogs
- reusable engine code
- cross-pool references
- compact unmanaged data
- deterministic runtime behavior

Together with `EntityHandle`, it forms the basis of the engine's typed entity architecture.