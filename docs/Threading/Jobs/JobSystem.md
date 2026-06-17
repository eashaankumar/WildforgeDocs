# JobSystem

Namespace:

```csharp
using Wildforge.Engine.Core.Threading.Jobs;
```

The Wildforge Job System is the engine's parallel execution layer for running high-performance work over unmanaged data.

It is designed for:

- dense entity updates
- simulation systems
- physics batches
- AI batches
- animation jobs
- terrain processing
- render extraction
- command recording

It is **not** intended to be a general-purpose task library like `Task`, `ThreadPool`, or `Parallel.For`.

---

# Purpose

Wildforge's engine architecture is built around dense unmanaged data.

Examples:

```text
UnmanagedList<T>
EntityPool<T>
UnmanagedView<T>
EntityPoolView<T>
```

The job system exists to process that data in parallel without introducing managed allocations or GC pressure during gameplay.

A typical job looks like:

```text
Process range 0..1023

Process range 1024..2047

Process range 2048..3071
```

Each worker receives a range of indices and operates on that range.

---

# Why Not Task?

`Task` is convenient, but it is not the right abstraction for Wildforge's hot path.

Problems:

- managed allocations
- scheduler overhead
- unpredictable timing
- general-purpose behavior
- difficult deterministic control
- not designed around unmanaged views

Wildforge needs a job system that is explicitly designed for engine workloads.

---

# Why Not Parallel.For?

`Parallel.For` is also convenient, but it is still a high-level managed abstraction.

For tools or tests, it may be fine.

For the engine's core simulation loop, Wildforge should own:

- worker lifetime
- batch size
- scheduling policy
- command-buffer integration
- memory layout
- synchronization rules

The job system should be predictable and engine-specific.

---

# Core Concept

A job is an unmanaged struct containing the data needed to execute work.

```csharp
public unsafe struct DinoUpdateJob : IJob
{
    public EntityPool<Dino>.EntityPoolView<Dino> Dinos;

    public float DeltaTime;

    public void Execute(
        int start,
        int count,
        int workerIndex)
    {
        for (int i = start; i < start + count; i++)
        {
            ref var dino = ref Dinos[i];

            dino.Position += dino.Velocity * DeltaTime;
        }
    }
}
```

The job system divides the work into batches and runs those batches across worker threads.

---

# Job Interface

A minimal job interface looks like:

```csharp
public interface IJob
{
    void Execute(
        int start,
        int count,
        int workerIndex);
}
```

Each call receives:

```text
start

count

workerIndex
```

The job processes:

```text
[start, start + count)
```

---

# Why Range-Based Jobs?

Range-based jobs are ideal for dense data.

Example:

```text
Dino array

0 1 2 3 4 5 6 7 8 9
```

The job system can split this into:

```text
worker 0: 0..2
worker 1: 3..5
worker 2: 6..9
```

Each worker writes a separate range.

This avoids locks.

---

# Passing Data Into Jobs

A job can contain multiple unmanaged fields.

Example:

```csharp
public unsafe struct DinoSimulationJob : IJob
{
    public EntityPool<Dino>.EntityPoolView<Dino> Dinos;

    public UnmanagedView<Plant> Plants;

    public UnmanagedView<BiomeData> Biomes;

    public WorkerCommandBuffer* Commands;

    public float DeltaTime;

    public int FrameIndex;
}
```

Jobs may contain:

- pointers
- unmanaged views
- entity views
- primitive values
- settings structs
- command buffer pointers
- fixed-size unmanaged state

Jobs may not contain:

- strings
- managed arrays
- delegates
- classes
- `List<T>`
- `Dictionary<TKey,TValue>`
- managed references

---

# Views Instead Of Owners

Jobs should receive views, not owning containers.

Good:

```csharp
public UnmanagedView<Dino> Dinos;
```

or:

```csharp
public EntityPool<Dino>.EntityPoolView<Dino> Dinos;
```

Avoid:

```csharp
public UnmanagedList<Dino> Dinos;
```

or:

```csharp
public EntityPool<Dino> Dinos;
```

The owning containers expose structural operations such as:

- resize
- add
- remove
- clear
- dispose

Jobs should not have access to those operations.

Views allow jobs to read and write existing memory without modifying structure.

---

# Multiple Inputs

Jobs can receive as many inputs as needed.

Example:

```csharp
public unsafe struct ProjectileJob : IJob
{
    public EntityPool<Projectile>.EntityPoolView<Projectile> Projectiles;

    public EntityPool<Dino>.EntityPoolView<Dino> Dinos;

    public EntityPool<Mammal>.EntityPoolView<Mammal> Mammals;

    public UnmanagedView<TerrainCell> Terrain;

    public WorkerCommandBuffer* Commands;

    public float DeltaTime;
}
```

This is normal.

The job is just a struct.

---

# Writing To Ranges

A job may write to the range it owns.

Safe:

```csharp
for (int i = start; i < start + count; i++)
{
    Dinos[i].Health -= 1;
}
```

Unsafe:

```csharp
Dinos[0].Health -= 1;
```

from every worker.

The engine assumes jobs follow range ownership rules.

---

# Worker Index

The `workerIndex` identifies the worker executing the batch.

This is primarily useful for accessing worker-local data.

Example:

```csharp
ref var commands =
    ref Commands[workerIndex];
```

Each worker writes only to its own command buffer.

---

# Worker-Local Command Buffers

Jobs should not structurally mutate the world.

Bad:

```csharp
world.Dinos.TryDestroy(handle);
```

Good:

```csharp
Commands[workerIndex].DestroyDinos.TryAdd(
    new DestroyDinoCommand
    {
        Handle = handle
    });
```

After all jobs finish:

```text
worker command buffers

↓

merge

↓

ordered apply
```

This preserves structural safety.

---

# Structural Rules

During jobs, systems may:

- read existing data
- write assigned ranges
- write worker-local command buffers
- write worker-local scratch memory

During jobs, systems may not:

- create entities directly
- destroy entities directly
- resize containers
- clear containers
- dispose containers
- write overlapping ranges without synchronization

---

# Batch Size

Batch size controls how many items each job batch processes.

Small batch size:

```text
more scheduling overhead
better load balancing
```

Large batch size:

```text
less scheduling overhead
worse load balancing
```

Typical starting values:

```text
128
256
512
1024
```

The correct value depends on the work per entity.

Light jobs usually need larger batches.

Heavy jobs can use smaller batches.

---

# Persistent Workers

The job system should create worker threads once.

```text
Engine start

↓

Create workers

↓

Reuse every frame

↓

Shutdown
```

Do not create threads every frame.

Thread creation is expensive and managed.

---

# No Per-Dispatch Managed Allocation

The job system should avoid managed allocation during dispatch.

This means:

- no `Task` allocation
- no captured lambdas
- no per-job delegate allocations
- no managed queues per frame

The system may own managed thread objects created during initialization.

That is acceptable.

The hot dispatch path should avoid GC.

---

# Job Data Lifetime

The job struct must remain valid for the duration of the job.

Do not pass pointers to stack data that may go out of scope before workers finish.

Usually the scheduler blocks until all batches complete:

```csharp
jobSystem.ScheduleParallel(
    ref job,
    count,
    batchSize);
```

If scheduling becomes asynchronous later, job lifetime rules must become stricter.

---

# Determinism

The job system can be deterministic if:

- batch ranges are deterministic
- merge order is deterministic
- command apply order is deterministic
- jobs avoid data races

Parallel execution order itself may vary, so systems should not rely on worker execution order.

---

# Example: Updating Dinos

```csharp
public unsafe struct DinoUpdateJob : IJob
{
    public EntityPool<Dino>.EntityPoolView<Dino> Dinos;

    public float DeltaTime;

    public void Execute(
        int start,
        int count,
        int workerIndex)
    {
        int end = start + count;

        for (int i = start; i < end; i++)
        {
            ref var dino = ref Dinos[i];

            dino.Position += dino.Velocity * DeltaTime;

            dino.Hunger += DeltaTime;
        }
    }
}
```

Usage:

```csharp
var job = new DinoUpdateJob
{
    Dinos = world.Dinos.AsView(),
    DeltaTime = deltaTime
};

jobSystem.ScheduleParallel(
    ref job,
    world.Dinos.Count,
    batchSize: 512);
```

---

# Example: Recording Destroy Commands

```csharp
public unsafe struct DinoDeathJob : IJob
{
    public EntityPool<Dino>.EntityPoolView<Dino> Dinos;

    public EntityHandle* Handles;

    public WorkerCommandBuffer* Commands;

    public void Execute(
        int start,
        int count,
        int workerIndex)
    {
        ref var commands =
            ref Commands[workerIndex];

        int end = start + count;

        for (int i = start; i < end; i++)
        {
            ref var dino = ref Dinos[i];

            if (dino.Health <= 0)
            {
                commands.DestroyDinos.TryAdd(
                    new DestroyDinoCommand
                    {
                        Handle = Handles[i]
                    });
            }
        }
    }
}
```

This pattern is safe because structural mutation is deferred.

---

# Relationship To EntityPool

`EntityPool<T>` provides dense entity memory.

The job system processes that memory.

```text
EntityPool<T>

↓

AsView()

↓

Job

↓

Parallel Update
```

The pool should not structurally change while a job is reading its view.

---

# Relationship To Commands

Jobs record commands.

Commands mutate the world later.

```text
Job

↓

Worker Command Buffer

↓

Merge

↓

Apply Commands

↓

World Mutated
```

This keeps parallel simulation safe.

---

# Relationship To UnmanagedView

`UnmanagedView<T>` is the general-purpose view type for contiguous unmanaged memory.

Jobs should prefer views because views do not expose ownership.

```csharp
public UnmanagedView<TerrainCell> Terrain;
```

This avoids accidental structural mutation.

---

# Failure Handling

Jobs should generally avoid throwing exceptions.

Engine runtime code should prefer:

- validation before scheduling
- debug assertions
- error counters
- command failure results

Throwing from worker threads complicates shutdown and recovery.

---

# Safety

The job system is powerful but unsafe.

It assumes:

- views remain valid
- ranges do not overlap
- unmanaged memory is not disposed mid-job
- jobs do not mutate shared state unsafely

In debug builds, the engine may eventually add validation.

Release builds should remain lean.

---

# Best Practices

## Do

Use views.

Use worker-local command buffers.

Use deterministic merge order.

Preallocate command buffers.

Choose batch size carefully.

Keep jobs focused.

Avoid managed references.

---

## Don't

Do not resize containers inside jobs.

Do not destroy entities inside jobs.

Do not share one command buffer across workers.

Do not capture managed closures.

Do not rely on execution order.

Do not pass invalid pointers.

---

# Design Philosophy

The Wildforge job system is not a general task framework.

It is specifically for:

```text
unmanaged data

+

range-based parallel execution

+

worker-local command recording
```

This focused design keeps it smaller, faster, and easier to reason about.

---

# Summary

The Wildforge Job System is the parallel execution layer for dense unmanaged engine data.

It exists to process large arrays, entity pools, simulation data, terrain chunks, and render extraction workloads with minimal overhead and no per-dispatch GC allocation.

It works together with:

- `UnmanagedView<T>`
- `EntityPool<T>.AsView()`
- `WorldCommandList<TWorld,TCommand>`
- worker-local command buffers
- deferred structural mutation

to form the foundation of Wildforge's high-performance simulation model.