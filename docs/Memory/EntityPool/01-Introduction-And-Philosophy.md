# EntityPool<T>: Introduction and Philosophy

Namespace:

```csharp
using Wildforge.Engine.Core.TypedEntityPools;
```

`EntityPool<T>` is Wildforge's dense unmanaged storage container for runtime entities.

```csharp
public unsafe struct EntityPool<T> : IDisposable
    where T : unmanaged
```

It is designed for game entities that are naturally processed as complete objects:

- dinosaurs
- mammals
- plants
- projectiles
- items
- world objects
- simulation agents

Wildforge deliberately does **not** treat every entity as a loose bag of tiny components. A dinosaur is stored as a dinosaur. A plant is stored as a plant. A projectile is stored as a projectile.

This gives systems direct access to the complete runtime object:

```csharp
var view = world.Dinos.AsView();

for (int i = 0; i < view.Count; i++)
{
    ref var dino = ref view[i];
    UpdateDino(ref dino);
}
```

## Why Typed Entity Pools?

Traditional ECS architectures split objects into many separate component arrays:

```text
Transform[]
Velocity[]
AIState[]
AnimationState[]
Health[]
Inventory[]
```

That can be powerful, but it also forces systems to repeatedly gather related data.

For Forgewild-style simulation, many systems naturally touch most of the creature:

- AI reads state, health, transform, nearby targets, requests, hunger, stamina.
- Animation reads movement state, skeleton state, velocity, actions.
- Physics writes transform-linked data.
- Rendering reads transform, graphics, animation, palette and state.
- Gameplay logic mutates many fields together.

So Wildforge favors cohesive typed objects stored in dense unmanaged arrays.

## Design Goals

`EntityPool<T>` is built around these goals:

- unmanaged storage
- no GC-tracked entity memory
- stable runtime handles
- dense contiguous iteration
- O(1) creation
- O(1) destruction
- swap-remove deletion
- stale-handle protection
- predictable performance
- compatibility with parallel simulation
- compatibility with command buffers
- no Unity dependency
- no ECS component fragmentation

## What EntityPool Is

`EntityPool<T>` is:

- an owner of entity memory
- a stable handle allocator
- a dense iterator source
- a lifetime validator
- a runtime storage primitive

## What EntityPool Is Not

`EntityPool<T>` is not:

- a database
- a global entity registry
- a serializer
- a scheduler
- a spatial index
- an ECS archetype
- a thread-safe structural mutation container

Other systems build on top of it.

## Typical Usage

```csharp
var dinos = new EntityPool<Dino>(1024);

dinos.TryCreate(new Dino(), out var handle);

if (dinos.TryGet(handle, out var ptr))
{
    ptr->Health -= 10;
}

dinos.TryDestroy(handle);

dinos.Dispose();
```

## The Big Idea

Externally, code uses stable handles.

Internally, entities stay packed densely.

That means the pool can offer both:

```text
stable references
+
fast contiguous iteration
```

This combination is the heart of the design.
