# RTMaterial

`RTMaterial` is the CPU-side material instance for ray tracing hit groups.

Current milestone:

- Selects a hit group from an `RTShader`.
- Stores named material values.
- If the hit group has a reflected `ShaderStructLayout`, property names and basic value types are validated when set.
- `PackMaterialData()` produces a zero-initialized byte buffer matching the reflected struct size and writes all set values at reflected offsets.

This does **not** upload data to the GPU yet. The next milestone is an `RTMaterialStore` that assigns material indices, packs many material instances into GPU buffers, and exposes those buffers to reflected descriptor binding.
