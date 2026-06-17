# UnmanagedQueue<T>

A high-performance unmanaged FIFO queue.

`UnmanagedQueue<T>` stores its elements inside unmanaged memory using a circular buffer.

`T` must be unmanaged.

---

## Declaration

```csharp
public unsafe struct UnmanagedQueue<T> : IDisposable
    where T : unmanaged
```

---

## Constructors

### UnmanagedQueue(int capacity)

Creates a queue.

```csharp
var queue = new UnmanagedQueue<Job>(32);
```

---

## Properties

### Count

Current number of elements.

```csharp
int count = queue.Count;
```

### Capacity

Current buffer capacity.

```csharp
int capacity = queue.Capacity;
```

---

## Methods

### TryEnqueue()

Adds an element to the back of the queue.

Returns `false` if insertion fails.

```csharp
queue.TryEnqueue(job);
```

---

### TryDequeue()

Removes the front element.

```csharp
if (queue.TryDequeue(out var job))
{
}
```

---

### TryPeek()

Returns the front element without removing it.

```csharp
if (queue.TryPeek(out var job))
{
}
```

---

### Clear()

Removes all queued elements while preserving allocated memory.

```csharp
queue.Clear();
```

---

### Dispose()

Releases unmanaged memory.

```csharp
queue.Dispose();
```

---

## Performance

| Operation | Complexity |
|------------|-----------:|
| Enqueue | O(1) |
| Dequeue | O(1) |
| Peek | O(1) |
| Resize | O(n) |

---

## Notes

- Uses a circular buffer internally.
- Automatically grows when capacity is exceeded.
- Allocates only unmanaged memory.
- Ideal for job queues, event systems, and simulation pipelines.