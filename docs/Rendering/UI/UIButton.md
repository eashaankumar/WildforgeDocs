# UIButton

Namespace:

```csharp
using Wildforge.Engine.Core.Rendering.UI;
```

`UIButton` is an interactive UI control that responds to mouse input.

It automatically tracks hover and pressed states, changes its appearance based on those states, and raises events when the user interacts with it.

`UIButton` inherits from `UIElement`.

---

# Purpose

Use `UIButton` whenever the player should be able to click on something.

Typical uses include:

- Main menu buttons
- Inventory buttons
- Toolbar buttons
- Settings controls
- Shop items
- Dialog choices

---

# Inheritance

```text
UIElement

↓

UIButton
```

---

# Button States

Every button is always in one of three states.

```csharp
public enum State
{
    Normal,
    Hover,
    Pressed
}
```

The current state can be read through:

```csharp
public State CurrentState { get; }
```

---

## Normal

The mouse is not over the button.

```
┌──────────────┐
│    Button    │
└──────────────┘
```

---

## Hover

The mouse is inside the button.

```
┌══════════════┐
║    Button    ║
└══════════════┘
```

---

## Pressed

The left mouse button is currently being held after pressing inside the button.

```
┏━━━━━━━━━━━━━━┓
┃    Button    ┃
┗━━━━━━━━━━━━━━┛
```

---

# Textures

Buttons support three independent textures.

## TextureNormal

```csharp
public Texture2D TextureNormal;
```

Displayed while the button is in the normal state.

---

## TextureHover

```csharp
public Texture2D TextureHover;
```

Displayed while the mouse is hovering over the button.

---

## TexturePressed

```csharp
public Texture2D TexturePressed;
```

Displayed while the button is pressed.

---

# Material

```csharp
public RasterMaterial Material;
```

The material used to render the button.

The material should expose:

```
MainTexture

Tint
```

These values are updated automatically before rendering.

---

# Events

Buttons expose several events describing the user's interaction.

---

## Clicked

```csharp
public event Action<UIButton>? Clicked;
```

Raised when:

1. the left mouse button is pressed inside the button
2. the mouse is released while still inside the button

Example:

```csharp
button.Clicked += button =>
{
    Console.WriteLine("Play");
};
```

---

## MouseEnter

```csharp
public event Action<UIButton>? MouseEnter;
```

Raised when the cursor first enters the button.

Example:

```csharp
button.MouseEnter += _ =>
{
    PlayHoverSound();
};
```

---

## MouseExit

```csharp
public event Action<UIButton>? MouseExit;
```

Raised when the cursor leaves the button.

---

## MouseDown

```csharp
public event Action<UIButton>? MouseDown;
```

Raised when the left mouse button is pressed while the cursor is inside the button.

---

## MouseUp

```csharp
public event Action<UIButton>? MouseUp;
```

Raised when the left mouse button is released while the cursor is inside the button.

---

# RectTransform

Like every UI element, a button uses a `RectTransform`.

Example:

```csharp
button.RectTransform = new RectTransform
{
    Position = new Vector3(400, 300, 0),
    Size = new Vector2(240, 64),
    Scale = Vector3.One,
    Rotation = Quaternion.Identity,
    Pivot01 = new Vector2(0.5f, 0.5f)
};
```

The transform controls:

- position
- size
- rotation
- scale
- pivot

---

# Creating A Button

```csharp
var button = new UIButton
{
    TextureNormal = normalTexture,
    TextureHover = hoverTexture,
    TexturePressed = pressedTexture,
    Material = uiMaterial,

    RectTransform = new RectTransform
    {
        Position = new Vector3(400, 300, 0),
        Size = new Vector2(240, 64),
        Scale = Vector3.One,
        Rotation = Quaternion.Identity,
        Pivot01 = new Vector2(0.5f, 0.5f)
    }
};

canvas.AddElement(button);
```

---

# Handling Clicks

Subscribe to the `Clicked` event.

```csharp
button.Clicked += _ =>
{
    StartGame();
};
```

This is the primary way of responding to button presses.

---

# Hover Effects

Hover events are useful for:

- playing sounds
- tooltips
- highlighting
- animations

Example:

```csharp
button.MouseEnter += _ =>
{
    hoverSound.Play();
};

button.MouseExit += _ =>
{
    tooltip.Hide();
};
```

---

# Mouse Down / Up

Sometimes a control should react immediately instead of waiting for a full click.

Example:

```csharp
button.MouseDown += _ =>
{
    pressedSound.Play();
};

button.MouseUp += _ =>
{
    releasedSound.Play();
};
```

---

# Click Behavior

A click only occurs when:

```
Mouse Down

↓

Inside Button

↓

Mouse Up

↓

Still Inside Button
```

If the user presses inside the button but releases outside it:

```
No Click
```

This matches the behavior of most desktop user interfaces.

---

# Rendering

The button automatically chooses the correct texture.

```
Normal

↓

TextureNormal
```

```
Hover

↓

TextureHover
```

```
Pressed

↓

TexturePressed
```

The selected texture is assigned to the material before rendering.

---

# Sorting

Like every `UIElement`, buttons inherit:

```csharp
public int SortingOrder;
```

Higher values appear above lower values.

Example:

```csharp
panel.SortingOrder = 0;

button.SortingOrder = 10;

text.SortingOrder = 20;
```

---

# Example

```csharp
var playButton = new UIButton
{
    TextureNormal = normal,
    TextureHover = hover,
    TexturePressed = pressed,
    Material = uiMaterial,

    RectTransform = new RectTransform
    {
        Position = new Vector3(500, 300, 0),
        Size = new Vector2(240, 64),
        Scale = Vector3.One,
        Rotation = Quaternion.Identity,
        Pivot01 = new Vector2(0.5f, 0.5f)
    }
};

playButton.Clicked += _ =>
{
    Console.WriteLine("Play pressed");
};

playButton.MouseEnter += _ =>
{
    Console.WriteLine("Hover");
};

canvas.AddElement(playButton);
```

---

# Typical Menu

```
Canvas

↓

Background

↓

Play Button

↓

Options Button

↓

Quit Button
```

Each button independently tracks its own mouse interaction.

---

# Best Practices

## Do

Provide textures for all three button states.

Use `Clicked` for primary interaction.

Use hover events for optional feedback.

Reuse a shared UI material.

---

## Don't

Do not call `Update()` or `Render()` directly.

Do not manually change `CurrentState`.

Do not rely on polling mouse input instead of subscribing to events.

---

# Relationship To Other Types

```
UICanvas

↓

UIButton

↓

Texture2D

RasterMaterial

RectTransform

Input
```

The canvas owns the button.

The engine `Input` object drives interaction.

The `RectTransform` determines the clickable region.

---

# Summary

`UIButton` is Wildforge's built-in clickable UI control.

It automatically handles mouse interaction, visual state changes, and click events while integrating naturally with the engine's `UICanvas`, `Input`, and rendering systems.