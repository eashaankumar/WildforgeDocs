# IntervalThread

Namespace:

```csharp
using Wildforge.Engine.Core.Threading;
```

`IntervalThread` is a small threading utility for running work repeatedly on its own thread at a fixed interval.

It is intended for **low-frequency background work**, not hot-path simulation.

---

# Purpose

Use `IntervalThread` when you need a dedicated loop that wakes up periodically, performs some work, then sleeps again.

Typical uses:

- diagnostics
- autosave checks
- editor refresh loops
- background polling
- log flushing
- watchdog-style checks
- lightweight service loops

Do **not** use `IntervalThread` for entity simulation, physics, AI, rendering extraction, or per-frame gameplay systems.

For high-performance parallel simulation, use the Wildforge job system instead.

---

# Concept

An interval thread behaves like this:

```text
Start thread

↓

while running:

    execute callback

    sleep interval

↓

stop thread
```

The important feature is that the work happens outside the main thread.

---

# Typical Usage

```csharp
var thread = new IntervalThread(
    intervalMilliseconds: 1000,
    action: () =>
    {
        Console.WriteLine("Runs once per second.");
    });

thread.Start();

// Later:
thread.Dispose();
```

This creates a thread that runs the callback roughly once every second.

---

# When To Use

`IntervalThread` is appropriate when the work:

- does not need to run every frame
- does not need exact timing
- can safely happen in the background
- is independent from hot simulation loops
- is simple enough to run on one dedicated thread

Good examples:

```text
Check autosave timer every few seconds

Flush logs periodically

Update editor diagnostics

Poll a debug service

Run a slow monitoring loop
```

---

# When Not To Use

Do not use `IntervalThread` for:

- updating entities
- physics
- animation
- AI
- chunk meshing
- terrain generation
- render extraction
- command-buffer processing
- high-frequency jobs

Those systems should use the job system or dedicated engine pipelines.

---

# Timing Behavior

`IntervalThread` is interval-based, not frame-perfect.

If the interval is:

```text
1000 ms
```

then the callback will run approximately once per second.

However, exact timing depends on:

- operating system scheduling
- callback duration
- thread wake-up timing
- CPU load

This should not be used for deterministic simulation timing.

---

# Callback Duration

The callback time adds to the total loop time unless the implementation compensates for it.

For example:

```text
callback takes 50 ms

interval is 1000 ms
```

The actual gap between callback starts may be slightly more than one second depending on implementation.

Therefore, callbacks should be short.

---

# Thread Ownership

`IntervalThread` owns one managed thread.

That thread exists until the interval thread is stopped or disposed.

This is different from the job system, which owns a pool of workers and executes many small jobs.

---

# Lifetime

Typical lifetime:

```text
Create

↓

Start

↓

Run repeatedly

↓

Dispose
```

Always dispose the thread when it is no longer needed.

```csharp
thread.Dispose();
```

Disposal should stop the loop and release thread resources.

---

# Start

Starting begins execution.

```csharp
thread.Start();
```

After `Start`, the callback runs periodically until the thread is stopped/disposed.

---

# Stop / Dispose

Stopping should end the worker loop.

Disposing should stop the thread and clean up resources.

```csharp
thread.Dispose();
```

Prefer `Dispose` when the object is no longer needed.

---

# Thread Safety

The callback runs on a background thread.

That means it must not directly mutate data owned by the main thread unless synchronization is used.

Unsafe example:

```csharp
// Background thread directly mutates game world.
world.Dinos.TryCreate(new Dino(), out _);
```

Better:

```csharp
// Background thread records or signals work.
// Main thread applies changes later.
```

If the callback touches shared state, protect that state using an appropriate synchronization strategy.

---

# Engine State Access

Be careful accessing engine state.

Safe patterns include:

- reading immutable config
- writing to a thread-safe log queue
- setting atomic flags
- recording requests for the main thread
- using locks around small shared data

Unsafe patterns include:

- mutating entity pools
- changing render state
- modifying command buffers shared by other threads
- accessing disposed memory

---

# Relationship To JobSystem

`IntervalThread` and the job system solve different problems.

## IntervalThread

```text
Dedicated background loop

Low frequency

Service-style work

One callback repeatedly
```

## JobSystem

```text
Parallel execution

High frequency

Frame work

Many jobs over dense data
```

Use `IntervalThread` for background services.

Use the job system for simulation.

---

# Example: Autosave Check

```csharp
var autosaveThread = new IntervalThread(
    intervalMilliseconds: 5000,
    action: () =>
    {
        autosaveRequested = true;
    });
```

The main thread can later observe the flag and perform the actual save safely.

---

# Example: Log Flushing

```csharp
var logFlushThread = new IntervalThread(
    intervalMilliseconds: 1000,
    action: () =>
    {
        logger.Flush();
    });
```

This keeps log flushing out of the main game loop.

---

# Example: Editor Diagnostics

```csharp
var diagnosticsThread = new IntervalThread(
    intervalMilliseconds: 250,
    action: () =>
    {
        diagnostics.Update();
    });
```

Useful for tools that do not need frame-perfect updates.

---

# Design Notes

## Small Utility

`IntervalThread` is intentionally simple.

It is not a scheduler.

It is not a job system.

It is not a task graph.

It is simply a repeating background thread.

---

## Managed Thread

Because it wraps a thread, this is not a fully unmanaged system.

That is acceptable because `IntervalThread` is not intended for hot loops.

The high-performance simulation path should use the job system instead.

---

## Explicit Lifetime

The caller owns the interval thread and must dispose it.

This avoids hidden global background workers.

---

# Best Practices

## Do

Use it for low-frequency background tasks.

Keep callbacks short.

Dispose it when finished.

Avoid direct world mutation from the callback.

Use thread-safe communication.

---

## Don't

Do not use it for per-frame gameplay.

Do not use it for entity simulation.

Do not use it for physics or rendering.

Do not perform long blocking work without understanding its impact.

Do not access unmanaged memory that may be disposed by another thread.

---

# Summary

`IntervalThread` is Wildforge's simple utility for repeating background work at a fixed interval.

It is useful for services, tools, diagnostics, and periodic checks.

It should not be confused with the job system.

For simulation and high-performance parallel processing, use Wildforge's job architecture instead.