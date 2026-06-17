# EntityPool<T> API Reference

## Properties

```csharp
int Count
```

Number of alive entities.

```csharp
int Capacity
```

Current capacity of dense entity storage.

```csharp
int SlotCount
```

Number of allocated slots.

```csharp
int FreeCount
```

Number of reusable dead slots.

```csharp
uint StructuralVersion
```

Version incremented when pool structure changes.

```csharp
bool IsCreated
```

Whether the pool owns allocated memory.

## Construction

```csharp
new EntityPool<T>(int initialCapacity)
```

Creates a pool with initial entity and slot capacity.

## Capacity

```csharp
TryEnsureCapacity(int entityCapacity)
TryEnsureSlotCapacity(int slotCapacity)
TryEnsureTotalCapacity(int entityCapacity, int slotCapacity)
TrySetEntityCapacity(int entityCapacity)
TrySetSlotCapacity(int slotCapacity)
TryTrimExcess()
```

## Creation

```csharp
TryCreate(in T value, out EntityHandle handle)
TryCreate(out EntityHandle handle)
TryCreate(out EntityHandle handle, out T* ptr)
```

## Lookup

```csharp
IsAlive(EntityHandle handle)
TryGetDenseIndex(EntityHandle handle, out int denseIndex)
TryGet(EntityHandle handle, out T* ptr)
UnsafeGet(EntityHandle handle)
TryGetByDenseIndex(int denseIndex, out T* ptr)
UnsafeGetByDenseIndex(int denseIndex)
TryGetHandleByDenseIndex(int denseIndex, out EntityHandle handle)
UnsafeGetHandleByDenseIndex(int denseIndex)
```

## Destruction

```csharp
TryDestroy(EntityHandle handle)
UnsafeDestroyAlive(EntityHandle handle)
```

## Views

```csharp
AsView()
```

Returns a dense pointer/count view.

## Lifetime

```csharp
Clear()
Dispose()
```

`Clear()` removes logical entities but retains memory.

`Dispose()` frees unmanaged memory.

## Safety Summary

Use `TryX` methods in normal code.

Use `UnsafeX` methods only in carefully validated hot paths.

## Example

```csharp
var pool = new EntityPool<Dino>(1024);

pool.TryCreate(out var handle, out var dino);

dino->Health = 100;

if (pool.TryGet(handle, out var ptr))
{
    ptr->Health -= 5;
}

pool.TryDestroy(handle);

pool.Dispose();
```
