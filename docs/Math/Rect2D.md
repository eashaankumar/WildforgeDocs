# Rect2D

`Rect2D` represents a 2D rectangle using a center point and size.

## Purpose

Use `Rect2D` for:

-   UI hit testing
-   2D bounds
-   Selection rectangles
-   Sprite bounds
-   Broad-phase collision
-   Viewport calculations

## Construction

``` csharp
var rect = new Rect2D(
    new Vector2(0,0),
    new Vector2(100,50));
```

Or:

``` csharp
var rect = Rect2D.CreateRect2DFromMinMax(min,max);
```

## Properties

-   Center
-   Size
-   Min
-   Max
-   Left
-   Right
-   Bottom
-   Top

## Operations

``` csharp
rect.Contains(point);
rect.Intersects(other);
```

Containment and intersection are inclusive of edges.

## Design

The rectangle stores **Center + Size**. Min/Max are computed, preventing
duplicated state and keeping mutations simple.

## Example

``` csharp
if(rect.Contains(mousePosition))
{
    // Hovered
}
```
