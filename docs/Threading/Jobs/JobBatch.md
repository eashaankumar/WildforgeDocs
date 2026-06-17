# JobBatch

`JobBatch` represents one scheduled unit of work executed by the Wildforge Job System.

A batch does **not** represent an entire job.

Instead, it represents a **portion** of a job.

For example:

```text
Entire Job

Update 100,000 dinos
```

may be divided into:

```text
Batch 0

0..1023
```

```text
Batch 1

1024..2047
```

```text
Batch 2

2048..3071
```

...

Each worker executes one batch at a time until all batches complete.

---

# Purpose

The purpose of batching is to distribute work across multiple CPU cores.

Instead of assigning one huge job to one thread:

```text
100,000 entities

↓

1 thread
```

the scheduler creates many smaller batches:

```text
100,000 entities

↓

196 batches

↓

8 workers
```

Workers continuously pull batches until none remain.

---

# Why Not One Batch Per Worker?

Suppose you have:

```text
8 workers
```

and:

```text
100,000 entities
```

One approach would be:

```text
Worker 0

12,500
```

```text
Worker 1

12,500
```

...

Unfortunately workloads are rarely uniform.

Example:

```text
Worker 0

Easy AI
```

```text
Worker 1

Boss AI
```

Worker 1 finishes much later.

Seven cores sit idle.

---

# Smaller Batches

Instead:

```text
512 entities

↓

batch
```

Workers continuously pull new batches.

```text
Worker 0

Batch 0

↓

Batch 8

↓

Batch 15
```

```text
Worker 1

Batch 1

↓

Batch 9

↓

Batch 16
```

The workload naturally balances itself.

---

# Batch Contents

Conceptually a batch contains:

```text
Job

↓

Start

↓

Count
```

Example:

```text
Job

↓

Start = 4096

Count = 512
```

The worker executes:

```csharp
job.Execute(
    4096,
    512,
    workerIndex);
```

---

# Batch Size

Batch size controls the amount of work contained in one batch.

Example:

```text
100,000 entities

Batch Size = 512
```

Produces approximately:

```text
196 batches
```

Changing batch size changes scheduler behavior.

---

# Choosing Batch Size

Small batches:

```text
better load balancing

more scheduling overhead
```

Large batches:

```text
less scheduling overhead

poorer balancing
```

Typical starting values:

```text
128

256

512

1024
```

The optimal size depends entirely on the cost of processing one element.

---

# Dynamic Scheduling

Wildforge uses dynamic scheduling.

Workers repeatedly request another batch.

Example:

```text
Worker

↓

Acquire Batch

↓

Execute

↓

Acquire Next Batch

↓

Execute

↓

Repeat
```

Workers stop only when no batches remain.

This prevents idle CPU cores.

---

# Worker Independence

Every batch is independent.

A worker should not need to know:

- previous batch
- next batch
- other workers

It only receives:

```text
start

count

workerIndex
```

Everything else comes from the job.

---

# Range Ownership

Each batch owns exactly one contiguous range.

Safe:

```text
Batch A

0..511
```

```text
Batch B

512..1023
```

Unsafe:

```text
Batch A

0..511
```

```text
Batch B

256..767
```

Overlapping writes create data races.

The scheduler guarantees non-overlapping ranges.

---

# Cache Locality

Contiguous ranges improve cache efficiency.

Instead of:

```text
0

50

100

150

...
```

workers process:

```text
512

513

514

515

516
```

Sequential memory access is significantly faster.

---

# Batch Lifetime

Typical lifecycle:

```text
Scheduler Creates Batch

↓

Worker Executes Batch

↓

Batch Discarded
```

Batches are temporary scheduling objects.

They should not persist beyond one dispatch.

---

# Relationship To Jobs

One job may produce many batches.

```text
Job

↓

Scheduler

↓

Batch

Batch

Batch

Batch
```

Each batch calls the same `Execute()` function with different ranges.

---

# Relationship To Workers

Workers do not own jobs.

Workers execute batches.

```text
Worker

↓

Acquire Batch

↓

Execute

↓

Acquire Batch

↓

Execute
```

This separation allows excellent load balancing.

---

# Performance

Creating batches is inexpensive.

A batch is conceptually just:

```text
Start

Count
```

No entity data is copied.

No views are duplicated.

Workers simply receive different ranges into the same data.

---

# Public vs Internal

Most users never interact directly with `JobBatch`.

Instead they schedule work through:

```csharp
jobSystem.ScheduleParallel(
    ref job,
    itemCount,
    batchSize);
```

The scheduler creates batches internally.

`JobBatch` is public primarily to document how work is partitioned, not because users are expected to construct batches themselves.

---

# Best Practices

## Do

Choose a reasonable batch size.

Assume batches execute in arbitrary order.

Keep batches independent.

Process only the assigned range.

---

## Don't

Do not communicate between batches.

Do not assume batch execution order.

Do not write outside the assigned range.

Do not perform structural world mutations.

---

# Summary

A `JobBatch` is one partition of a larger job.

It enables Wildforge to distribute large workloads efficiently across multiple worker threads while preserving:

- cache locality
- deterministic range ownership
- dynamic load balancing
- minimal scheduling overhead

Most games will never create `JobBatch` instances directly—the scheduler handles batching automatically.