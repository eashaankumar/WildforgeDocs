# ShaderBuilder

Compiles raster and raytracing piplines into spv and reflections.json.

## Raster

Input:

```c
*.rasterpipline
*.vert.glsl
*.frag.glsl
```

Output:

```c
*.reflection.json
*.vert.spv
*.frag.spv
```

## Raytracing

Input

```c
*.rtpipeline
*.rchit.glsl
*.rgen.glsl
*.rmiss.glsl
```

Output

```c
*.reflection.json
*.rchit.spv
*.rgen.spv
*.rmiss.spv
```

## Usage

Compilation:

```c
.\Tools\Forgewild.ShaderBuilder.exe <Folder OR .rt/.rasterpipeline>
```
