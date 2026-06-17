# IJob

`IJob` is the public contract implemented by jobs that run through Wildforge's job system.

A job is a small unmanaged struct that performs work over a range of data.

```csharp
public interface IJob
{
    void Execute(int start, int count, int workerIndex);
}
```

---

# Purpose

Use `IJob` when you want to run work in parallel over dense unmanaged data.

Typical examples:

- update all dinos
- update all projectiles
- process terrain cells
- run AI batches
- prepare render data
- write worker-local commands

---

# Basic Example

```csharp
public unsafe struct AddHealthJob : IJob
{
    public UnmanagedView<Dino> Dinos;
    public int Amount;

    public void Execute(int start, int count, int workerIndex)
    {
        int end = start + count;

        for (int i = start; i < end; i++)
        {
            Dinos[i].Health += Amount;
        }
    }
}
```

---

# Execute Parameters

## start

The first index this job batch owns.

## count

The number of elements this batch should process.

## workerIndex

The index of the worker executing the batch.

Use this for worker-local data:

```csharp
ref var commands = ref Commands[workerIndex];
```

---

# Range Ownership

A job should only write inside its assigned range.

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

---

# Job Data

Jobs are structs and can contain multiple unmanaged fields:

```csharp
public unsafe struct DinoJob : IJob
{
    public UnmanagedView<Dino> Dinos;
    public UnmanagedView<Plant> Plants;
    public WorkerCommandBuffer* Commands;

    public float DeltaTime;
    public int Frame;
}
```

Jobs should not contain managed references such as:

- `string`
- `List<T>`
- arrays
- classes
- delegates
- dictionaries

---

# Views Instead Of Owners

Jobs should receive views.

Good:

```csharp
public UnmanagedView<Dino> Dinos;
```

Avoid:

```csharp
public UnmanagedList<Dino> Dinos;
```

Views expose memory without exposing structural mutation.

---

# Structural Rule

Jobs may:

- read existing data
- write assigned ranges
- write worker-local command buffers

Jobs may not:

- create entities directly
- destroy entities directly
- resize containers
- clear containers
- dispose memory

Structural changes should be recorded as commands and applied later.

---

# Summary

`IJob` is the simplest possible contract for range-based parallel work.

It keeps Wildforge jobs:

- unmanaged
- explicit
- fast
- predictable
- compatible with dense memory
- compatible with worker-local command buffers