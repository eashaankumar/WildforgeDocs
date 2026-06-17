# Job Safety

Wildforge's Job System is designed around one simple principle:

> **Jobs process existing data. They do not change the structure of the world.**

This rule is what allows thousands or even millions of entities to be processed safely across many CPU cores without locks.

---

# The Two Phases

Every frame consists of two distinct phases.

```text
Simulation Phase

↓

Structural Phase
```

## Simulation Phase

Jobs execute in parallel.

They may:

- read entity data
- modify existing entity data
- write worker-local command buffers
- read terrain
- read world state

They may **not** change the structure of the world.

---

## Structural Phase

After every job has completed:

```text
Worker Command Buffers

↓

Merge

↓

Apply Commands
```

Only during this phase may the world:

- create entities
- destroy entities
- resize pools
- update spatial indexes
- rebuild handles
- perform structural mutations

---

# Why Structural Changes Are Dangerous

Suppose four workers are updating dinosaurs.

```text
Worker 0

Dinos 0..999
```

```text
Worker 1

Dinos 1000..1999
```

Worker 0 destroys:

```text
Dino 200
```

The entity pool performs swap-remove.

Now:

```text
Dense storage changes.
```

Worker 1 is still iterating.

Its view is now incorrect.

This can lead to:

- skipped entities
- duplicate updates
- invalid pointers
- crashes
- corrupted simulation

---

# The Correct Pattern

Instead of destroying immediately:

```csharp
world.Dinos.TryDestroy(handle);
```

record the intent:

```csharp
commands.DestroyDinos.TryAdd(
    new DestroyDinoCommand
    {
        Handle = handle
    });
```

Later:

```text
Merge

↓

Apply

↓

Destroy
```

No worker ever observes changing storage.

---

# Entity Views Are Immutable

When a job begins:

```csharp
var view = world.Dinos.AsView();
```

that view represents a snapshot of the pool's storage.

The underlying entity values may change.

The storage itself must not.

This means:

Allowed:

```text
Modify Health

Modify Position

Modify Velocity

Modify AI State
```

Not allowed:

```text
Resize Pool

Destroy Entity

Create Entity

Dispose Pool
```

---

# Worker Ownership

Each worker owns a range.

Example:

```text
Worker 0

0..511
```

```text
Worker 1

512..1023
```

Worker 0 may modify:

```text
0..511
```

It should never modify:

```text
512..1023
```

unless synchronization is explicitly used.

---

# Shared Reads

Multiple workers may safely read the same data.

Example:

```text
Terrain

↓

Worker 0

Worker 1

Worker 2

Worker 3
```

Reading immutable data requires no synchronization.

---

# Shared Writes

Writing shared memory is only safe when:

- ranges do not overlap
- synchronization exists
- the algorithm is specifically designed for it

Otherwise data races occur.

---

# Worker-Local Data

Whenever possible, workers should write only to data they exclusively own.

Examples:

```text
Worker-local command buffer
```

```text
Worker-local scratch allocator
```

```text
Worker-local statistics
```

This completely avoids contention.

---

# Command Buffers

Structural changes should always be expressed as commands.

Example:

```text
Kill Dinosaur

↓

DestroyDinoCommand
```

```text
Spawn Projectile

↓

SpawnProjectileCommand
```

```text
Attach Rider

↓

AttachRiderCommand
```

Commands are merged after simulation.

---

# Container Safety

Owning containers should never be modified during job execution.

Unsafe:

```csharp
list.TryAdd(...);

list.Clear();

list.Dispose();

pool.TryCreate(...);

pool.TryDestroy(...);
```

Safe:

```csharp
view[i].Health--;

view[i].Position += velocity;
```

---

# Views

Jobs should operate on views.

Good:

```csharp
UnmanagedView<T>

EntityPoolView<T>
```

Avoid passing owning containers.

Views intentionally expose:

- element access
- count

They do not expose structural operations.

---

# Job Lifetime

The memory referenced by a job must remain valid until every worker has completed.

Never dispose:

- pools
- unmanaged lists
- frame allocators
- views

while jobs are still executing.

The scheduler must guarantee completion before cleanup.

---

# Determinism

Parallel execution order is undefined.

Jobs must never rely on:

```text
Worker 0 finishes first
```

or

```text
Batch 3 executes before Batch 4
```

Instead, correctness must depend only on the assigned range.

---

# Avoid Global State

Jobs should avoid writing shared global variables.

Bad:

```csharp
TotalHealth += dino.Health;
```

Good:

```csharp
WorkerTotals[workerIndex] += dino.Health;
```

Later:

```text
Reduce Worker Totals

↓

Final Total
```

---

# False Sharing

Even when workers write different values, writing nearby cache lines can reduce performance.

Prefer:

```text
Worker-local memory
```

instead of:

```text
Shared array of counters
```

unless the array is padded appropriately.

---

# Unsafe APIs

Wildforge intentionally exposes low-level APIs.

The engine assumes callers obey the rules.

Release builds prioritize performance over defensive runtime checks.

Debug builds may introduce additional validation in the future.

---

# Safety Checklist

Before scheduling a job, verify:

- ✓ Views remain valid for the duration of the job
- ✓ Workers write only to their assigned ranges
- ✓ Structural changes are deferred as commands
- ✓ Shared writes are synchronized or avoided
- ✓ Worker-local buffers are independent
- ✓ No containers will be disposed during execution

---

# Best Practices

## Do

Use views.

Use worker-local command buffers.

Keep batches independent.

Record structural changes instead of applying them.

Reuse worker-local memory.

Treat jobs as temporary.

---

## Don't

Do not resize containers.

Do not create or destroy entities.

Do not write overlapping ranges.

Do not depend on execution order.

Do not dispose memory while jobs are running.

Do not mutate shared state without synchronization.

---

# Summary

Wildforge's job safety model is intentionally simple:

> **Jobs process data. Command buffers change the world.**

By separating simulation from structural mutation, the engine achieves:

- deterministic behavior
- lock-free entity iteration
- stable views
- excellent cache locality
- scalable parallel execution

This philosophy is one of the core design principles behind the Wildforge engine.