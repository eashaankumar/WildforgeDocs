# UIElement

Namespace:

```csharp
using Wildforge.Engine.Core.Rendering.UI;
```

`UIElement` is the base class for every user interface object in Wildforge.

All built-in UI controls derive from `UIElement`, including:

- `UISprite`
- `UIText`
- `UIButton`

You can also derive from `UIElement` to create your own custom controls.

---

# Purpose

A `UIElement` represents a single object that can participate in the UI system.

Each element owns:

- a transform
- a render order
- optional update logic
- optional rendering logic

A canvas manages collections of UI elements.

---

# Inheritance

```text
UIElement
│
├── UISprite
├── UIText
└── UIButton
```

Custom controls should also inherit from `UIElement`.

---

# Creating A Custom Element

Derive from `UIElement` and override the methods you need.

```csharp
public class HealthBar : UIElement
{
    public override void Update(
        Input input,
        RectTransform rootTransform)
    {
        // update logic
    }

    public override void Render(
        RasterPass pass)
    {
        // rendering logic
    }
}
```

---

# RectTransform

Every UI element owns a `RectTransform`.

```csharp
public RectTransform RectTransform;
```

The `RectTransform` controls:

- Position
- Rotation
- Scale
- Size
- Pivot

Example:

```csharp
element.RectTransform = new RectTransform
{
    Position = new Vector3(100, 50, 0),
    Rotation = Quaternion.Identity,
    Scale = Vector3.One,
    Size = new Vector2(200, 64),
    Pivot01 = new Vector2(0.5f, 0.5f)
};
```

The transform determines where the element appears on the canvas.

---

# SortingOrder

```csharp
public int SortingOrder;
```

`SortingOrder` controls render order.

Lower values render first.

Higher values render later.

Example:

```text
SortingOrder = 0

↓

Background
```

```text
SortingOrder = 10

↓

Button
```

```text
SortingOrder = 20

↓

Text
```

Later elements appear on top of earlier ones.

---

# Update

```csharp
public virtual void Update(
    Input input,
    RectTransform rootTransform)
```

Override this to implement interaction or animation.

Example:

```csharp
public override void Update(
    Input input,
    RectTransform rootTransform)
{
    if (input.IsKeyPressed(KeyCode.Space))
    {
        // perform action
    }
}
```

The canvas calls `Update()` once every frame.

---

# Render

```csharp
public virtual void Render(
    RasterPass pass)
```

Override this to draw the element.

Example:

```csharp
public override void Render(
    RasterPass pass)
{
    // draw custom UI
}
```

Rendering happens after the canvas updates all elements.

---

# Canvas Membership

A `UIElement` is inactive until it is added to a `UICanvas`.

Example:

```csharp
canvas.AddElement(element);
```

Removing the element:

```csharp
canvas.RemoveElement(element);
```

The canvas owns update order and render order.

---

# Lifetime

Typical lifetime:

```text
Create

↓

Configure RectTransform

↓

Add To Canvas

↓

Updated Every Frame

↓

Rendered Every Frame

↓

Remove From Canvas
```

---

# Coordinate System

A `UIElement` uses its `RectTransform` for positioning.

The transform is evaluated relative to the canvas.

This allows every UI element to use a consistent positioning model.

---

# Custom Controls

Creating custom controls is straightforward.

Example:

```csharp
public class InventorySlot : UIElement
{
    public Item Item;

    public override void Render(
        RasterPass pass)
    {
        // draw slot
    }
}
```

The engine does not require registration or reflection.

Any class deriving from `UIElement` can be used.

---

# Best Practices

## Do

Use `RectTransform` for layout.

Override only the methods you need.

Keep rendering separate from update logic.

Use `SortingOrder` to control layering.

Add elements through `UICanvas`.

---

## Don't

Do not call `Render()` yourself.

Do not call `Update()` yourself.

Do not modify canvas collections while iterating them.

Do not use `SortingOrder` as a substitute for layout.

---

# Relationship To Other Types

```text
UICanvas

↓

UIElement

↓

UISprite
UIText
UIButton
```

The canvas owns UI elements.

UI elements define behavior and rendering.

Concrete UI controls inherit from `UIElement`.

---

# Summary

`UIElement` is the foundation of Wildforge's UI system.

It provides:

- a `RectTransform`
- render ordering
- per-frame updates
- rendering hooks

Every built-in and custom UI control derives from `UIElement`.