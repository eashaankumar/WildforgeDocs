# Entity Kind

User defines their own kinds

```csharp
public static class GameEntityKinds
{
    public static readonly EntityKind Dino = new(1);
    public static readonly EntityKind Mammal = new(2);
    public static readonly EntityKind Shrimp = new(3);
    public static readonly EntityKind Projectile = new(4);
    public static readonly EntityKind Plant = new(5);
    public static readonly EntityKind Item = new(6);
    public static readonly EntityKind StaticObject = new(7);
}
```

## Usage

```csharp
var target = new EntityRef(
    GameEntityKinds.Dino,
    dinoHandle);
```

## Why this is better than enum

An enum forces the engine to know game types:

```csharp
public enum EntityKind
{
    Dino,
    Mammal,
    Plant
}
```

Bad for a reusable engine.

A value-type ID lets each game define its own entity catalog while the engine remains generic.