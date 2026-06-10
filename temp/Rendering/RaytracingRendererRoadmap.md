# Ray tracing hello triangle

The renderer target is **not** a raster hello triangle.

First visible milestone:

1. SDL3 creates a Vulkan-capable window and surface.
2. Vulkan device is selected only if it supports ray tracing extensions.
3. A single triangle vertex buffer is uploaded.
4. BLAS is built from that triangle.
5. TLAS is built with one instance referencing the BLAS.
6. Raygen shader traces one ray per pixel into the TLAS.
7. Closest-hit shader colors the triangle with barycentric color.
8. Miss shader writes a dark background.
9. Raygen writes into a storage image.
10. Storage image is copied/blitted into the swapchain image and presented.

No raster graphics pipeline, no vertex shader, no fragment shader, and no `vkCmdDraw` are part of this milestone.

## Shader files

- `Core/Rendering/Vulkan/Shaders/RtTriangle.rgen`
- `Core/Rendering/Vulkan/Shaders/RtTriangle.rmiss`
- `Core/Rendering/Vulkan/Shaders/RtTriangle.rchit`
