# Input System

Public user-facing documentation for the Wildforge input module.

This document describes only the APIs gameplay/user code should call. Internal implementation details, helper classes, SDL mapper classes, and analog-keyboard HID internals are intentionally not part of this public surface.

---

## Purpose

`Input` is the engine's unified input object. It lets game code read keyboard, mouse, gamepad, and supported analog-keyboard input through one API.

The design goal is simple:

```csharp
float moveX = input.GetAxis(Axes.MoveX);
```

The gameplay code should not need to know whether that axis came from:

- normal keyboard keys,
- analog keyboard keys,
- mouse buttons,
- controller sticks,
- controller triggers.

The source is chosen when the axis is registered.

---

## Public API Overview

### `Input`

Create one `Input` instance for a window and call `Update()` once per frame.

```csharp
var input = new Input(window);

while (running)
{
    input.Update();

    // game input reads here
}

input.Dispose();
```

Public members:

```csharp
public Vector2 MousePosition { get; }
public Vector2 MouseDelta { get; }
public float MouseScrollDeltaX { get; }
public float MouseScrollDeltaY { get; }

public bool HasGamepad { get; }

public void RegisterAxis(...);
public float GetAxis(int axisId);

public Vector2 GetGamepadLeftStick(float deadzone = 0.12f);
public Vector2 GetGamepadRightStick(float deadzone = 0.12f);
public float GetGamepadAxis(GamepadAxis axis, float deadzone = 0.12f);

public float GetAnalogKeyDepth(KeyCode key);

public bool IsKeyDown(KeyCode key);
public bool IsKeyPressed(KeyCode key);
public bool IsKeyReleased(KeyCode key);

public bool IsMouseButtonDown(MouseButton button);
public bool IsMouseButtonPressed(MouseButton button);
public bool IsMouseButtonReleased(MouseButton button);

public void Dispose();
```

---

## Key Concepts

### Digital keyboard input

Digital keyboard input reports keys as `pressed-or-not-pressed` values.

```csharp
if (input.IsKeyDown(KeyCode.Space))
{
    player.JumpHeld();
}

if (input.IsKeyPressed(KeyCode.Space))
{
    player.JumpStarted();
}

if (input.IsKeyReleased(KeyCode.Space))
{
    player.JumpReleased();
}
```

Use digital key APIs for menus, UI, shortcuts, hotkeys, one-shot actions, and anything that should behave like a normal button.

---

## Frame Updates

Call `Update()` exactly once per frame.

```csharp
while (running)
{
    input.Update();

    // Read input here.
}
```

Reading input before calling `Update()` may return values from the previous frame.

---

## Axis Registration

Axes are registered by integer ID. The engine/game can define those IDs however it wants.

Example:

```csharp
public static class Axes
{
    public const int MoveX = 0;
    public const int MoveY = 1;
    public const int LookX = 2;
    public const int LookY = 3;
    public const int MenuX = 4;
}
```

Then register bindings:

```csharp
input.RegisterAxis(
    Axes.MoveX,
    minKey: KeyCode.A,
    maxKey: KeyCode.D);

input.RegisterAxis(
    Axes.MoveY,
    minKey: KeyCode.S,
    maxKey: KeyCode.W);
```

Read them:

```csharp
var move = new Vector2(
    input.GetAxis(Axes.MoveX),
    input.GetAxis(Axes.MoveY));
```

Axis values are normalized to `-1..1`:

```text
negative side only -> -1
nothing            ->  0
positive side only ->  1
```

For analog-capable sources, partial values are possible:

```text
half-pressed A -> -0.5
half-pressed D ->  0.5
```

### Axis Range

Every registered axis returns a normalized value in the range:

```text
-1.0 ... 1.0
```

Values are automatically clamped to this range before being returned by `GetAxis`.

---

## `KeyAxisMode`

Keyboard-backed axes can choose how key values should be interpreted.

```csharp
public enum KeyAxisMode
{
    Digital,
    Analog
}
```

### `KeyAxisMode.Digital`

Digital mode treats keys as hard buttons.

```text
not pressed -> 0
pressed     -> 1
```

Use this for menus, UI, hotbars, inventory navigation, shortcuts, and any input where partial pressure should not matter.

```csharp
input.RegisterAxis(
    Axes.MenuX,
    minKey: KeyCode.A,
    maxKey: KeyCode.D,
    keyAxisMode: KeyAxisMode.Digital);
```

### `KeyAxisMode.Analog`

Analog mode asks the analog-keyboard system for a floating-point key depth when available.

```text
not pressed      -> 0.0
partially pressed -> between 0.0 and 1.0
fully pressed     -> 1.0
```

Use this for movement, vehicle throttle, steering, leaning, walk/run pressure, or any action where partial key travel should matter.

```csharp
input.RegisterAxis(
    Axes.MoveX,
    minKey: KeyCode.A,
    maxKey: KeyCode.D,
    keyAxisMode: KeyAxisMode.Analog);
```

If the key has not been learned yet by the analog-keyboard system, or if the user has a normal keyboard, analog mode falls back to digital behavior. This means analog axes still work on ordinary keyboards.

---

## Same Keys, Different Axis Modes

You can register two axes with the same physical keys but different `KeyAxisMode` values.

```csharp
input.RegisterAxis(
    Axes.MoveX,
    minKey: KeyCode.A,
    maxKey: KeyCode.D,
    keyAxisMode: KeyAxisMode.Analog);

input.RegisterAxis(
    Axes.MenuX,
    minKey: KeyCode.A,
    maxKey: KeyCode.D,
    keyAxisMode: KeyAxisMode.Digital);
```

This is allowed because `KeyAxisMode` belongs to the axis registration, not to the key itself.

So the same key can behave like this:

```text
MoveX: analog movement depth
MenuX: hard left/right menu navigation
```

---

## Analog Keyboard Support

Analog keyboard support is intentionally hidden behind the public `Input` API.

User/game code does **not** create or manage an analog-keyboard object directly. It uses:

```csharp
input.GetAnalogKeyDepth(KeyCode.W);
```

or registers an axis with:

```csharp
keyAxisMode: KeyAxisMode.Analog
```

### Why this design exists

Normal keyboard APIs report keys as booleans: pressed or not pressed. Analog keyboards can report key travel depth, but there is no universal public API shared by all keyboard vendors.

So `Input` exposes a vendor-neutral public API:

```csharp
float depth = input.GetAnalogKeyDepth(KeyCode.W);
```

The engine internally handles supported analog keyboards and maps their device-specific key reports onto engine `KeyCode`s.

### What value does `GetAnalogKeyDepth` return?

```csharp
float w = input.GetAnalogKeyDepth(KeyCode.W);
```

Possible values:

```text
0.0  -> key is not pressed
0.25 -> key is pressed lightly
0.75 -> key is pressed deeply
1.0  -> key is fully pressed, or digital fallback is active
```

If analog data is unavailable, the method falls back to digital keyboard state:

```text
key up   -> 0.0
key down -> 1.0
```

This fallback is important because gameplay code can safely use the method without checking what hardware the player owns.

---

## Gamepad Input

Gamepad axes are exposed through `GamepadAxis`.

```csharp
public enum GamepadAxis
{
    LeftX,
    LeftY,
    RightX,
    RightY,
    LeftTrigger,
    RightTrigger
}
```

Read sticks directly:

```csharp
Vector2 move = input.GetGamepadLeftStick();
Vector2 look = input.GetGamepadRightStick();
```

Read one gamepad axis:

```csharp
float throttle = input.GetGamepadAxis(GamepadAxis.RightTrigger);
```

Register a whole axis from a gamepad axis:

```csharp
input.RegisterAxis(
    Axes.LookX,
    gamepadAxis: GamepadAxis.RightX);
```

Register a combined trigger axis:

```csharp
input.RegisterAxis(
    Axes.Rotate,
    minGamepadAxis: GamepadAxis.LeftTrigger,
    maxGamepadAxis: GamepadAxis.RightTrigger);
```

This produces:

```text
left trigger  -> negative value
right trigger -> positive value
none          -> 0
```

### Trigger Values

Gamepad triggers return values in the range:

```text
0.0 ... 1.0
```

where:

```text
0.0 -> released

1.0 -> fully pressed
```

Thumbsticks instead return:

```text
-1.0 ... 1.0
```

allowing movement in both positive and negative directions.

### No Connected Gamepad

If no supported gamepad is connected:

```csharp
input.HasGamepad
```

returns:

```text
false
```

Gamepad methods return neutral values.

For example:

```text
GetGamepadAxis(...)      -> 0

GetGamepadLeftStick()    -> (0,0)

GetGamepadRightStick()   -> (0,0)
```

This allows gameplay code to safely read gamepad input without first checking whether a controller is connected.

---

## Deadzones

Gamepad sticks and triggers support configurable deadzones.

The default deadzone is:

```csharp
0.12f
```

Input values inside the deadzone return zero.

For example:

```text
0.05 -> 0

0.10 -> 0

0.30 -> scaled output
```

Deadzones remove small hardware drift that commonly occurs with analog sticks.

Example:

```csharp
Vector2 move =
    input.GetGamepadLeftStick(
        deadzone: 0.20f);
```

The same deadzone parameter is also available when registering gamepad-backed axes.

---

## HasGamepad

```csharp
public bool HasGamepad { get; }
```

Returns whether a supported gamepad is currently connected.

Example:

```csharp
if (input.HasGamepad)
{
    ShowControllerHints();
}
else
{
    ShowKeyboardHints();
}
```

This value updates automatically as controllers are connected or disconnected.

---

## Mouse Input

Mouse state is exposed directly:

```csharp
Vector2 position = input.MousePosition;
Vector2 delta = input.MouseDelta;
float scrollY = input.MouseScrollDeltaY;
```

`MouseDelta` represents movement since the previous call to `Update()`.

Mouse buttons:

```csharp
if (input.IsMouseButtonDown(MouseButton.Left))
{
    // held
}

if (input.IsMouseButtonPressed(MouseButton.Left))
{
    // clicked this frame
}

if (input.IsMouseButtonReleased(MouseButton.Left))
{
    // released this frame
}
```

Mouse buttons can also contribute to axes:

```csharp
input.RegisterAxis(
    Axes.Zoom,
    minMouseButton: MouseButton.Right,
    maxMouseButton: MouseButton.Left);
```

### Mouse Coordinate System

Mouse coordinates use the engine's window coordinate system.

The origin is located at the bottom-left corner of the window.

```text
(0,0)

↓

Bottom Left
```

Increasing X moves right.

Increasing Y moves upward.

---

## Disposal

`Input` manages native resources associated with supported input devices.

When the input system is no longer needed, dispose it.

```csharp
input.Dispose();
```

Disposal releases resources associated with supported gamepads and analog keyboard devices.

Most applications create one `Input` instance for the lifetime of the main window and dispose it during shutdown.

---

## Recommended Usage Patterns

### Character movement with analog keyboard support

```csharp
input.RegisterAxis(
    Axes.MoveX,
    minKey: KeyCode.A,
    maxKey: KeyCode.D,
    keyAxisMode: KeyAxisMode.Analog);

input.RegisterAxis(
    Axes.MoveY,
    minKey: KeyCode.S,
    maxKey: KeyCode.W,
    keyAxisMode: KeyAxisMode.Analog);
```

Runtime:

```csharp
var move = new Vector2(
    input.GetAxis(Axes.MoveX),
    input.GetAxis(Axes.MoveY));

player.Move(move);
```

### Menu navigation with the same keys

```csharp
input.RegisterAxis(
    Axes.MenuX,
    minKey: KeyCode.A,
    maxKey: KeyCode.D,
    keyAxisMode: KeyAxisMode.Digital);
```

Runtime:

```csharp
if (input.GetAxis(Axes.MenuX) > 0)
{
    menu.MoveRight();
}
else if (input.GetAxis(Axes.MenuX) < 0)
{
    menu.MoveLeft();
}
```

### Controller movement

```csharp
input.RegisterAxis(
    Axes.MoveX,
    gamepadAxis: GamepadAxis.LeftX);

input.RegisterAxis(
    Axes.MoveY,
    gamepadAxis: GamepadAxis.LeftY);
```

Runtime:

```csharp
var move = new Vector2(
    input.GetAxis(Axes.MoveX),
    -input.GetAxis(Axes.MoveY));
```

Or directly:

```csharp
Vector2 move = input.GetGamepadLeftStick();
```

---

## Why Axis Registration Uses `min` and `max`

Keyboard and button axes are naturally made of two sides:

```text
A = negative side
D = positive side
```

So the registration API uses:

```csharp
minKey: KeyCode.A,
maxKey: KeyCode.D
```

The final value is:

```text
max side - min side
```

Examples:

```text
A only      -> 0 - 1 = -1
D only      -> 1 - 0 =  1
both        -> 1 - 1 =  0
neither     -> 0 - 0 =  0
```

For analog keys, the same math applies with floating point depth:

```text
A depth 0.25, D depth 0.75 -> 0.75 - 0.25 = 0.50
```

This is why digital and analog keyboard input can share the same axis code.

---

## Public vs Internal Surface

### Public / user-facing

Use these from gameplay code:

```csharp
Input
KeyCode
MouseButton
GamepadAxis
KeyAxisMode
```

Main public calls:

```csharp
input.Update();
input.RegisterAxis(...);
input.GetAxis(...);
input.GetAnalogKeyDepth(...);
input.GetGamepadAxis(...);
input.GetGamepadLeftStick(...);
input.GetGamepadRightStick(...);
input.IsKeyDown(...);
input.IsKeyPressed(...);
input.IsKeyReleased(...);
input.IsMouseButtonDown(...);
input.IsMouseButtonPressed(...);
input.IsMouseButtonReleased(...);
input.Dispose(...);
```

### Internal / not user-facing

Do not write gameplay code against the internal analog-keyboard implementation, SDL mapper helpers, or private axis storage. Those exist so `Input` can present a clean public API.

In particular, user code should not depend on vendor-specific analog keyboard details. Use `GetAnalogKeyDepth` or `KeyAxisMode.Analog` instead.

---

## Design Summary

The input system is built this way because it needs to support both ordinary input and newer analog devices without forcing game code to care about hardware details.

The important design choices are:

1. **Axes are source-agnostic.** Game code asks for an axis value, not a hardware device.
2. **Keyboard axes choose digital or analog behavior per axis.** The same key can be analog for movement and digital for menus.
3. **Analog keyboard support falls back to digital.** Games still work on normal keyboards.
4. **Gamepad input uses the same axis abstraction.** Controller sticks/triggers and keyboard axes can feed the same gameplay systems.
5. **Vendor-specific analog keyboard handling is internal.** Public code uses engine `KeyCode`s and float values only.
