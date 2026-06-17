# Creating Entities

`EntityPool<T>` provides two main creation styles.

## Create From Value

```csharp
pool.TryCreate(in value, out EntityHandle handle);
```

Example:

```csharp
var dino = new Dino
{
    Health = 100
};

if (world.Dinos.TryCreate(in dino, out var handle))
{
    // handle references the new dino
}
```

This copies the provided struct into dense pool storage.

## Create Default

```csharp
pool.TryCreate(out EntityHandle handle);
```

This creates a default `T`.

Useful when the entity will be filled immediately afterward.

## Create In Place

```csharp
pool.TryCreate(out EntityHandle handle, out T* ptr);
```

Example:

```csharp
if (world.Dinos.TryCreate(out var handle, out var dino))
{
    dino->Health = 100;
    dino->State = DinoState.Idle;
}
```

This is preferred for large structs.

It avoids creating a large temporary object and then copying it into the pool.

## Slot Allocation

Creation follows this logic:

```text
if free slot exists:
    reuse free slot
else:
    create new slot
```

Then:

```text
append entity data to _items
append slot id to _denseToSlot
mark slot alive
return handle
```

## Complexity

Creation is amortized O(1).

It can become O(n) only when unmanaged buffers grow and must copy existing memory to a new allocation.

## Preallocation

For best performance, create pools with expected capacity.

```csharp
var dinos = new EntityPool<Dino>(expectedDinoCount);
```

or reserve later:

```csharp
dinos.TryEnsureCapacity(10000);
dinos.TryEnsureSlotCapacity(10000);
```

Preallocation reduces resizing during gameplay.

## Recommended Pattern for Large Entities

Use in-place creation:

```csharp
if (pool.TryCreate(out var handle, out var ptr))
{
    FillDino(ptr, spawnData);
}
```

This is especially important for large gameplay structs such as dinosaurs.
