# UICanvas

Namespace:

```csharp
using Wildforge.Engine.Core.Rendering.UI;
```

`UICanvas` is the root container for a user interface.

It owns a collection of `UIElement`s, updates them every frame, and renders them in sorting order.

Every UI element must belong to a canvas before it can participate in the UI system.

---

# Purpose

A canvas is responsible for:

- storing UI elements
- updating UI interaction
- maintaining render order
- defining the root UI transform
- resizing to match the application window

Typical usage:

```csharp
var canvas = new UICanvas(
    input,
    new Vector2(window.Width, window.Height));
```

---

# Creating A Canvas

Construct a canvas by providing the engine `Input` object and the initial canvas size.

```csharp
var canvas = new UICanvas(
    input,
    new Vector2(1920, 1080));
```

The canvas internally stores the input object and creates a root `RectTransform`.

---

# RectTransform

Every canvas owns a root `RectTransform`.

```csharp
public RectTransform RectTransform;
```

This transform defines the coordinate system for every child UI element.

The default transform is:

```text
Position = (0,0,0)

Rotation = Identity

Scale = (1,1,1)

Pivot = (0,0)
```

Most applications never need to modify the canvas transform directly.

---

# Resize

```csharp
canvas.Resize(width, height);
```

Updates the canvas dimensions.

Example:

```csharp
canvas.Resize(
    window.Width,
    window.Height);
```

This should typically be called whenever the application window changes size.

The canvas updates its root `RectTransform.Size`.

---

# AddElement

```csharp
canvas.AddElement(element);
```

Adds a UI element to the canvas.

Example:

```csharp
canvas.AddElement(button);

canvas.AddElement(label);

canvas.AddElement(background);
```

When an element is added:

- it becomes part of the update loop
- it becomes part of the render loop
- the canvas automatically sorts its elements

---

# RemoveElement

```csharp
canvas.RemoveElement(element);
```

Removes an element from the canvas.

Example:

```csharp
canvas.RemoveElement(button);
```

After removal, the element no longer updates or renders.

---

# SortElements

```csharp
canvas.SortElements();
```

Sorts all UI elements by their `SortingOrder`.

Lower values render first.

Higher values render last.

Example:

```text
SortingOrder

0

↓

Background

10

↓

Button

20

↓

Text
```

Normally you do not need to call this manually because:

```csharp
AddElement()

RemoveElement()
```

already sort automatically.

Call it yourself only if you change an element's `SortingOrder` after it has already been added.

Example:

```csharp
button.SortingOrder = 100;

canvas.SortElements();
```

---

# Update

```csharp
canvas.Update();
```

Updates every UI element.

Typical frame:

```csharp
input.Update();

canvas.Update();
```

During update, each element receives:

- the current `Input`
- the canvas root transform

This allows controls such as `UIButton` to process mouse interaction.

---

# Rendering

Rendering is performed internally by the engine.

Applications generally do not call rendering methods directly.

The renderer traverses the canvas and renders every element in sorted order.

---

# Render Order

Elements render in ascending `SortingOrder`.

Example:

```text
SortingOrder = 0

↓

Panel
```

```text
SortingOrder = 5

↓

Icon
```

```text
SortingOrder = 10

↓

Text
```

Higher sorting orders always appear above lower ones.

---

# Canvas Ownership

One UI element should belong to only one canvas at a time.

Typical relationship:

```text
UICanvas

├── Background

├── Button

├── Label

└── Inventory
```

The canvas owns the update and rendering lifecycle of its elements.

---

# Window Resizing

A typical resize handler looks like:

```csharp
canvas.Resize(
    window.Width,
    window.Height);
```

This ensures the canvas coordinate system matches the application window.

---

# Typical Frame

Most applications follow this pattern:

```csharp
input.Update();

canvas.Update();

// Render frame
```

The renderer automatically draws the canvas later in the frame.

---

# Multiple Canvases

Applications may create multiple canvases.

Example:

```text
Main Menu

HUD

Pause Menu

Debug Overlay
```

Each canvas maintains its own collection of UI elements.

---

# Example

```csharp
var canvas = new UICanvas(
    input,
    new Vector2(1280, 720));

var background = new UISprite
{
    Texture = backgroundTexture,
    Material = spriteMaterial,
    SortingOrder = 0
};

var playButton = new UIButton
{
    TextureNormal = normal,
    TextureHover = hover,
    TexturePressed = pressed,
    Material = spriteMaterial,
    SortingOrder = 10
};

var title = new UIText
{
    FontAsset = font,
    Text = "Wildforge",
    FontSize = 42,
    Color = Color.White,
    SortingOrder = 20
};

canvas.AddElement(background);
canvas.AddElement(playButton);
canvas.AddElement(title);
```

During the game loop:

```csharp
input.Update();

canvas.Update();
```

---

# Best Practices

## Do

Resize the canvas when the window size changes.

Add UI through `AddElement()`.

Use `SortingOrder` for layering.

Reuse the same canvas throughout the application's lifetime.

---

## Don't

Do not manually call element `Render()`.

Do not manually call element `Update()`.

Do not modify the element collection while iterating it.

Do not create a new canvas every frame.

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

The canvas owns every UI element and manages its lifetime.

---

# Summary

`UICanvas` is the root object of Wildforge's UI system.

It provides:

- UI element ownership
- update management
- render ordering
- root coordinate system
- automatic sorting
- window resizing support

Every visible UI element belongs to a `UICanvas`.