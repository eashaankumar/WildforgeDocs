# Forgewild Typed Entity Pools

### Why I Chose This Instead of ECS

---

# Overview

Forgewild uses a custom architecture called **Typed Entity Pools**.

The design goal is simple:

> Store game entities in dense unmanaged arrays, access them by `ref`, and organize them by type instead of splitting them into hundreds of tiny ECS components.

This document explains:

* What Typed Entity Pools are
* Why they fit Forgewild
* Why they outperform ECS for this particular game
* What problems ECS solves
* Why those problems do not apply to Forgewild
* The exact structure of the world
* Future considerations

---

# Core Philosophy

A dinosaur is a dinosaur.

It is not:

```text
Transform
TailSegment
TailSegment
TailSegment
BodyPart
AIState
State
PhysicsBody
RequestQueue
AnimationState
```

spread across ten ECS component arrays.

Instead:

```csharp
struct Dino
{
    BepuPhysicsHandle Physics;

    DinoSkeleton Skeleton;

    DinoState State;
    DinoAIState AI;

    EntityRequest Request;
}
```

A dinosaur is a single logical object.

Most systems operate on the entire dinosaur.

Therefore the dinosaur should be stored as a single object.

---

# What Is A Typed Entity Pool?

A Typed Entity Pool is simply:

```csharp
NativeList<Dino> dinos;
NativeList<Mammal> mammals;
NativeList<Shrimp> shrimp;
NativeList<Projectile> projectiles;
```

Each pool stores one specific type.

Example:

```text
World
├── DinoPool
├── MammalPool
├── ShrimpPool
├── ProjectilePool
├── ItemPool
├── PlantPool
└── StaticObjectPool
```

Every pool contains dense contiguous memory.

Example:

```text
[Dino][Dino][Dino][Dino][Dino]
```

in memory.

This is called:

```text
AoS (Array of Structures)
```

---

# Why Dense Arrays Matter

The hot path of the game should look like:

```csharp
for (int i = 0; i < count; i++)
{
    ref var dino = ref dinos[i];

    UpdateDino(ref dino);
}
```

This is extremely cache friendly.

CPU prefetchers love this pattern.

Memory access becomes:

```text
Dino
Dino
Dino
Dino
Dino
```

with no indirection.

---

# Why Ref Matters

Never do this:

```csharp
var dino = dinos[i];
```

because that copies the entire struct.

Instead:

```csharp
ref var dino = ref dinos[i];
```

Now no copy occurs.

The CPU operates directly on the data inside the pool.

---

# Example Dino Structure

Example:

```csharp
struct Dino
{
    BepuPhysicsHandle Physics;

    Transform Body;

    Transform LeftArm;
    Transform RightArm;

    Transform LeftLeg;
    Transform RightLeg;

    Transform Tail1;
    Transform Tail2;
    Transform Tail3;
    Transform Tail4;

    DinoState State;
    DinoAIState AI;

    EntityRequest Request;
}
```

or later:

```csharp
struct DinoSkeleton
{
    Transform Body;

    FixedList16<Transform> Tail;
    FixedList8<Transform> Neck;

    FixedList4<Transform> Legs;
    FixedList4<Transform> Arms;
}
```

This is preferable because AI and rendering both operate on the entire dinosaur.

---

# Why Tiny ECS Components Are Bad For Forgewild

Consider:

```csharp
TransformComponent
TailSegmentComponent
BodyComponent
AIStateComponent
DinoStateComponent
RequestComponent
PhysicsComponent
```

Now every system must stitch everything back together.

Example:

```csharp
ref var transform = ref world.Get<Transform>(entity);
ref var ai = ref world.Get<AIState>(entity);
ref var state = ref world.Get<DinoState>(entity);
ref var physics = ref world.Get<Physics>(entity);
```

This creates:

```text
More lookups
More indirection
More complexity
More memory fragmentation
More joins
```

while providing no useful benefit.

The dinosaur was always a dinosaur.

The ECS simply exploded it into pieces.

---

# The Biggest ECS Misconception

Many developers assume:

```text
ECS = Faster
```

This is false.

The truth is:

```text
ECS is faster when systems operate on small subsets of data.
```

Example:

Physics only needs:

Transform + Velocity

AI only needs:

Position + State

Render only needs:

Transform + Mesh

In that case ECS wins.

---

# Why Forgewild Is Different

Most Forgewild systems operate on entire creatures.

Example:

AI:

```csharp
dino.AI
dino.State
dino.Request
dino.Transform
```

Animation:

```csharp
dino.Body
dino.Tail
dino.Legs
dino.Neck
```

Rendering:

```csharp
dino.Body
dino.Tail
dino.Legs
dino.Neck
```

Everything is used together.

Therefore storing everything together is optimal.

---

# Coarse ECS vs Tiny ECS

There are actually three approaches.

## Tiny ECS

```text
Transform
Velocity
Health
AI
Request
Physics
```

Many tiny components.

Bad for Forgewild.

---

## Coarse ECS

```csharp
struct DinoLogicData
{
    ...
}

struct DinoRenderData
{
    ...
}

struct DinoPhysicsData
{
    ...
}
```

This is much better.

---

## Typed Entity Pools

```csharp
struct Dino
{
    Logic
    Render
    Physics
}
```

This is the simplest solution.

And likely the fastest for Forgewild.

---

# Spawning And Destroying Entities

Dense arrays alone are not enough.

We need stable references.

---

# Dino Handles

Never expose raw array indices.

Instead:

```csharp
public struct DinoHandle
{
    public int Id;
    public int Generation;
}
```

---

# Slot Table

```csharp
struct DinoSlot
{
    public int DenseIndex;
    public int Generation;
    public bool Alive;
}
```

The handle points to a slot.

The slot points to the actual dense array location.

---

# Swap Remove

When deleting:

```text
Remove Dino #5
Move Last Dino Into Slot #5
count--
```

Example:

Before:

```text
0
1
2
3
4
5
6
7
```

Destroy:

```text
5
```

After:

```text
0
1
2
3
4
7
6
```

The pool stays packed.

Iteration remains fast.

---

# Why Swap Remove Is Important

Without it:

```text
Dead
Dead
Dead
Alive
Dead
Alive
```

The CPU constantly jumps over holes.

Performance suffers.

Packed arrays are critical.

---

# My Rebuttals To ECS

## Many Different Entity Types

Concern:

```text
ECS handles many entity types.
```

Response:

Simply create more pools.

```text
DinoPool
MammalPool
ShrimpPool
ProjectilePool
ItemPool
```

Problem solved.

---

## Generic Queries

Concern:

```text
ECS allows querying.
```

Response:

Use a dedicated spatial index.

Example:

```csharp
world.Spatial.Dinos.QueryRadius(...)
```

This is actually more specialized and faster.

---

## Editor / Inspector

Concern:

```text
ECS makes inspection easier.
```

Response:

Use persistent IDs.

```csharp
FastGuid
EntityRef
DinoHandle
```

No ECS required.

---

## Dynamic Components

Concern:

```text
Components can be added and removed.
```

Response:

Forgewild is state based.

Example:

```csharp
enum DinoState
{
    Idle,
    Hunting,
    Sleeping,
    Dead
}
```

Instead of adding/removing components:

```csharp
dino.State = DinoState.Sleeping;
```

Unused data simply remains dormant.

---

## Shared Systems

Concern:

```text
Code reuse.
```

Response:

Use helper functions.

Example:

```csharp
Movement.Update(...)
Combat.Update(...)
Perception.Update(...)
```

Both mammals and dinosaurs can call them.

No ECS required.

---

# Spatial Queries

AI will rely heavily on querying.

Use dedicated spatial structures:

```csharp
SpatialIndex<DinoHandle>
SpatialIndex<MammalHandle>
SpatialIndex<ItemHandle>
```

Then:

```csharp
world.Spatial.Dinos.QueryRadius(...)
```

returns nearby dinosaurs.

This is much more direct than ECS queries.

---

# Global Entity References

One thing worth keeping.

```csharp
public struct EntityRef
{
    public EntityKind Kind;
    public int Id;
    public int Generation;
}
```

Example:

```csharp
dino.AI.Target =
    new EntityRef(
        EntityKind.Mammal,
        id,
        generation);
```

This enables:

```text
Save/load
Networking
Selection
Editor tools
Cross-type references
```

without introducing ECS.

---

# NativeHashMap Warning

Do NOT store the primary entity data inside:

```csharp
NativeHashMap<FastGuid, Dino>
```

Hot path:

```csharp
dinosById[guid]
```

is slower than:

```csharp
ref var dino = ref dinos[index];
```

Use hash maps only for lookup tables:

```csharp
FastGuid -> DinoHandle
EntityRef -> Handle
```

Keep actual entity data in dense arrays.

---

# What This Architecture Is Called

Several names are technically correct.

## Data-Oriented Design (DOD)

Broad category.

```text
DOD
├── ECS
├── AoS
├── SoA
├── Pools
└── Archetypes
```

---

## Handle-Based Pool

Because entities use:

```text
Generational Handles
Swap Remove
Dense Arrays
```

---

## Array Of Structures (AoS)

Memory layout:

```text
[Dino][Dino][Dino][Dino]
```

---

## Typed Entity Pools

The name chosen for Forgewild.

```text
Typed Entity Pools
```

This is the preferred term for the project.

---

# Final Conclusion

For Forgewild:

```text
No ECS.
```

Instead:

```text
Typed Entity Pools
Dense NativeArrays / NativeLists
Generational Handles
Swap Remove
Persistent Entity IDs
Spatial Query Structures
Shared Helper Systems
State Machines
```

This architecture wins because:

```text
The game is object-centric.
The game is state-centric.
Creatures are strongly typed.
Systems operate on whole creatures.
Transforms are shared by most systems.
AI and rendering both need the same data.
```

ECS excels when entities are arbitrary collections of components.

Forgewild creatures are not arbitrary.

A dinosaur is always a dinosaur.

Therefore storing and processing it as a dinosaur is both simpler and faster.
