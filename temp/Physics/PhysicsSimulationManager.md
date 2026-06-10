# Physics Simulation Manager

## Overview

`PhysicsSimulationManager` partitions the infinite world into large
simulation chunks.

Each chunk owns:

* a dedicated BEPU v2 `Simulation`
* independent rigid body state
* independent broadphase
* independent contact graph

Chunks are identified by persistent `Vector3Int` coordinates.

World positions are converted into chunk coordinates using:

```csharp
chunkCoord = floor(worldPos / chunkSize)
```

The manager lazily creates simulations on demand.

Internally it uses:

* open-addressed hash lookup
* linear probing
* tombstone deletion
* dense active simulation iteration
* fixed-capacity storage
* zero-GC runtime lookup

---

# Features

* Infinite chunk coordinate space
* Lazy simulation creation
* Persistent chunk identifiers
* Amortized O(1) lookup
* Dense active simulation iteration
* Zero GC allocations during lookup
* Fixed-capacity storage
* Shared BEPU memory/resources per engine instance
* Independent chunk simulations

---

# Public API

```csharp
Simulation GetOrCreateSim(Vector3 worldPos)

Simulation GetSim(Vector3 worldPos)

void DestroySim(Vector3 worldPos)

void TimestepAll(float dt)

Simulation GetActiveSimulation(int activeIndex)

int ActiveCount { get; }
```

---

# Usage

## Create or Access Simulation

```csharp
Vector3 worldPos =
    new Vector3(1500f, 50f, -900f);

Simulation sim =
    engine.PhysicsSimulations
        .GetOrCreateSim(worldPos);
```

If the chunk simulation does not exist, it is created automatically.

---

## Access Existing Simulation

```csharp
Simulation sim =
    engine.PhysicsSimulations
        .GetSim(worldPos);

if (sim != null)
{
    sim.Timestep(1f / 60f);
}
```

Returns `null` if the chunk simulation has not been created.

---

## Destroy Simulation

```csharp
engine.PhysicsSimulations
    .DestroySim(worldPos);
```

This disposes the BEPU simulation and frees its internal pooled allocations.

---

# Active Simulation Iteration

The manager maintains a dense active simulation list for efficient stepping.

Example:

```csharp
for (int i = 0; i < physics.ActiveCount; i++)
{
    Simulation sim =
        physics.GetActiveSimulation(i);

    sim.Timestep(dt);
}
```

Or:

```csharp
physics.TimestepAll(dt);
```

Iteration cost scales with active simulations only.

Unused hash table slots are never traversed during stepping.

---

# Chunk Partitioning

Example chunk size:

```csharp
new Vector3(1024f, 1024f, 1024f)
```

Example mappings:

| World Position | Chunk Coord |
|----------------|-------------|
| `(0,0,0)`      | `(0,0,0)`   |
| `(500,0,0)`    | `(0,0,0)`   |
| `(1500,0,0)`   | `(1,0,0)`   |
| `(-1,0,0)`     | `(-1,0,0)`  |
| `(-1500,0,0)`  | `(-2,0,0)`  |

---

# Internal Architecture

The manager internally stores:

```csharp
Simulation[]
Vector3Int[]
byte[]

int[] activeSlots
int[] activeIndices
```

Where:

* `Simulation[]` stores BEPU simulations
* `Vector3Int[]` stores persistent chunk coordinates
* `byte[]` stores slot states
* `activeSlots[]` stores dense active slot indices
* `activeIndices[]` maps hash slots back into dense active iteration

The system uses:

* open-addressed hashing
* linear probing
* tombstone deletion
* swap-back dense removal

No dictionaries are used.

---

# Shared Resources

All simulations within a single engine instance share:

* `BufferPool`
* `PhysicsNarrowPhaseCallbacks`
* `PhysicsPoseIntegratorCallbacks`
* `SolveDescription`

These resources are owned by the engine instance and reused across all chunk simulations.

This minimizes memory overhead and improves cache locality.

---

# Lifetime

The manager is owned by the engine instance.

Initialization:

```csharp
Engine engine = new Engine(desc);
```

Shutdown:

```csharp
engine.Dispose();
```

Disposal destroys:

* all chunk simulations
* BEPU internal allocations
* shared physics resources

---

# Memory Model

## Runtime Allocations

Lookup operations allocate zero GC memory.

Operations:

* `GetSim`
* `GetOrCreateSim`
* `DestroySim`
* `TimestepAll`

perform no managed allocations.

---

## Capacity

The manager uses fixed-capacity storage.

Capacity represents:

```text
maximum simultaneously active physics chunks
```

NOT total world size.

Capacity is specified during initialization:

```csharp
new PhysicsSimulationManager(
    capacity: 1024,
    chunkSize: chunkSize,
    shared: shared);
```

The actual storage size is rounded to the next power of two internally.

---

# Simulation Independence

Each chunk simulation is fully independent.

This means:

* separate rigid body sets
* separate broadphases
* separate contact graphs
* separate constraint solving

Bodies cannot directly collide across chunk boundaries.

Cross-chunk interaction must be implemented manually if required.

---

# Important Notes

* Hashes are NOT persistent identifiers.
* `Vector3Int` chunk coordinates are the true persistent IDs.
* `Simulation` objects are independent.
* Capacity should represent loaded/active chunks only.
* Dense active iteration avoids scanning unused hash slots.
* Lookup is amortized O(1), not strictly guaranteed O(1).
