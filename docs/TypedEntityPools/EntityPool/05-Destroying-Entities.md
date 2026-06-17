# Destroying Entities

Entities are destroyed using:

```csharp
pool.TryDestroy(handle);
```

Example:

```csharp
if (world.Projectiles.TryDestroy(projectileHandle))
{
    // projectile was destroyed
}
```

If the handle is invalid, stale or already destroyed, `TryDestroy` returns `false`.

## Swap-Remove

Destruction uses swap-remove.

Suppose the pool contains:

```text
dense 0: A
dense 1: B
dense 2: C
dense 3: D
```

Destroy `B` at dense index 1.

The pool moves the last entity into the removed slot:

```text
dense 0: A
dense 1: D
dense 2: C
```

Then the count shrinks.

This is O(1).

## Order Is Not Preserved

Destroying entities changes dense order.

Do not rely on entity iteration order unless you explicitly sort or maintain another structure.

## Slot Reuse

After destruction:

```text
slot.Alive = 0
slot.DenseIndex = -1
slot.Generation++
slot pushed to free stack
```

The slot can be reused by a future entity.

## Stale Handles

When a destroyed slot is reused, old handles remain invalid because the generation no longer matches.

## UnsafeDestroyAlive

The unsafe version assumes the handle is valid and alive.

Use it only when validation has already been performed.

```csharp
pool.UnsafeDestroyAlive(handle);
```

Most game code should use `TryDestroy`.

## Structural Changes

Destroying an entity is a structural change.

Do not destroy entities while iterating a view in a simulation phase.

Instead, record a destroy command and apply it later.
