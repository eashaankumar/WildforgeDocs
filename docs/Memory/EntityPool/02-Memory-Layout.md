# EntityPool<T>: Memory Layout

`EntityPool<T>` internally owns four unmanaged containers:

```text
_items
_slots
_denseToSlot
_freeSlots
```

Each one has a specific job.

## _items

`_items` stores the actual entity data.

```text
_items

[Dino][Dino][Dino][Dino][Dino]
```

Only alive entities exist in `_items`.

There are no holes.

This is the memory systems iterate every frame.

## _slots

`_slots` maps stable handle IDs to dense entity locations.

Each slot contains:

```csharp
internal struct PoolSlot
{
    public int DenseIndex;
    public int Generation;
    public byte Alive;
}
```

A slot answers:

```text
For this handle ID,
where is the entity currently stored?
Is it alive?
Which generation is it?
```

## _denseToSlot

`_denseToSlot` maps from dense index back to slot ID.

```text
dense index -> slot id
```

This is required because entities move during swap-remove deletion.

When the last entity moves into a deleted entity's place, its slot must be updated.

## _freeSlots

`_freeSlots` stores reusable dead slot IDs.

When an entity is destroyed, its slot ID is pushed onto the free stack.

When a new entity is created, the pool first tries to reuse a free slot before creating a new one.

## Public Handle ID vs Internal Slot ID

Wildforge reserves handle ID `0` as invalid.

Therefore:

```text
public handle id = internal slot id + 1
internal slot id = public handle id - 1
```

Example:

```text
EntityHandle.Id = 1
internal slot id = 0
```

This makes:

```csharp
default(EntityHandle).IsValid == false
```

which is safer and more intuitive for users.

## Alive Entity Layout

Suppose the pool contains three entities.

```text
_items

dense 0: Dino A
dense 1: Dino B
dense 2: Dino C
```

Slots might look like:

```text
slot 0 -> dense 2, generation 0, alive
slot 1 -> dense 0, generation 0, alive
slot 2 -> dense 1, generation 0, alive
```

The dense order does not have to match handle order.

That is intentional.

## Why This Layout?

This layout allows three important properties at the same time:

1. Stable handles
2. Dense iteration
3. O(1) destruction

Most simple containers can provide only one or two of these.

`EntityPool<T>` provides all three.
