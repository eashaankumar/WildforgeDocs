# EntityHandle and Generations

`EntityHandle` is the stable runtime reference used to refer to entities stored inside an `EntityPool<T>`.

```csharp
public readonly struct EntityHandle
{
    public readonly int Id;
    public readonly int Generation;

    public bool IsValid => Id > 0;
}
```

## Id

`Id` identifies the slot.

It does **not** identify the dense index.

Dense indices can change when entities are destroyed.

## Generation

`Generation` identifies the lifetime of the slot.

When a slot is reused, its generation changes.

This prevents old handles from accidentally pointing to newly created entities.

## Why Generation Is Necessary

Without generations:

```text
Create Dino A in slot 5
Destroy Dino A
Create Dino B in slot 5
Old handle now points to Dino B
```

That is a serious bug.

With generations:

```text
Dino A handle: Id 5, Generation 0
Destroy Dino A
Slot generation becomes 1
Dino B handle: Id 5, Generation 1
Old handle: Id 5, Generation 0
```

The old handle fails validation.

## Invalid Handles

Wildforge reserves ID `0` as invalid.

This means:

```csharp
default(EntityHandle).IsValid == false
```

This is important because default structs appear naturally in arrays, fields and uninitialized memory.

## Handles Are Runtime References

`EntityHandle` is for runtime only.

It should not be used as a save-file identity.

For persistence, use a separate persistent ID system such as a GUID registry.

## EntityRef

`EntityHandle` only identifies something inside one pool.

To reference across pools, Wildforge uses `EntityRef`.

```csharp
public readonly struct EntityRef
{
    public readonly EntityKind Kind;
    public readonly EntityHandle Handle;
}
```

`EntityKind` tells the world which pool to look in.

`EntityHandle` tells that pool which entity to resolve.

## Rule

Never expose dense indices as long-lived references.

Use handles for identity.

Use dense indices only for temporary iteration.
