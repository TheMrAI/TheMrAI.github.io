---
layout: page
title: Import them meshes
date: "2026-04-23"
summary: "Import meshes using the Wavefront (.obj) format, add option for rendering wireframes and breaking the engine
by creating too much geometry."
draft:  true
---

While one can theoretically construct some assets within a game engine itself, apart from the simplest cases
that is wildly impractical. Managing the additional UV maps, normal maps, texture generation, not to mention
any animation is just a set of problems that need to be solved. Putting the mesh itself together alone is a complex
enough problem that whole software families have been written around it.
It suffices to say, there should be no good reason for adding direct support for these within the engine itself.
Simply use existing tools, generate some supported formats and import them into the engine and our problems are solved.

Now what should we support? Ideally something that can handle everything. For reference, what types of
[Imports & Exports](https://docs.blender.org/manual/en/latest/files/import_export/index.html)
does [Blender](https://docs.blender.org/manual/en/latest/index.html) support?

- Alembic - (Open source) Animated meshes stored with baked geometric results. It does not store any rigs or physics related data for
reproducing the animation, only the resulting animation itself.
- BVH - BioVision Motion Capture data. Hierarchical data and its animation.
- Universal Scene Descriptor - [USD](https://openusd.org/release/index.html) is for storing whole hierarchical scenes containing
cameras, meshes, lights, materials, etc..
- Stanford PLY - [PLY](https://paulbourke.net/dataformats/ply/) can store mesh data, some color properties, but does not store any
transformation or scene related hierarchical data. 
- STL - [STL](https://en.wikipedia.org/wiki/STL_(file_format)) only seems to contain, triangulated mesh data. No color, no materials,
just the triangles and their normal.
- FBX - [FBX](https://www.autodesk.com/products/fbx/overview) is an Autodesk format that handles complex scene data, with animations.
- glTF - [glTF](https://www.khronos.org/gltf/) is a Royalty-free specification by Khronos) for storing meshes, materials, cameras, textures, animations.
- Wavefront - [Wavefront OBJ](https://paulbourke.net/dataformats/obj/) files are for storing basic geometric data and basic material properties.

And surely there are many more formats out there in the wild, but for now this list is detailed enough. As a start, something simple should be
supported, without any complex machinery like scenes and animation. That narrows it down to PLY, STL and Wavefront OBJ.
PLY and STL seem to be a bit restrictive. PLY can only contain exactly one object, and STL can only contain mesh data.
Wavefront OBJ seems to be the winner. It has a decent specification thanks to [Paul Bourke](https://paulbourke.net/dataformats/obj/) it is simple,
supports more then the other two and can also handle basic material properties with the additional `.mtl` files.

With all those decisions out of the way, lets see our reference scene.

## The monkey, dragon and the teapot (walk into a bar)

The scene targeted for this chapter 

{{< figure src="reference_scene.png" title="Reference scene">}}

and rendered using wireframes

{{< figure src="reference_scene_wireframe.png" title="Wireframe reference scene">}}

These are the targets, but it should be noted that our wireframe scene will be different from the one in Godot. Simply because we will
not implement any sort of LOD (level of detail) optimization for the meshes. This is a similar idea to mipmaps. The further something
is, the crappier it can look and nobody will notice it, but the performance can increase drastically.
Godot internally generates the different levels of detail from a single mesh as it is being imported. These types of algorithms are not
in the scope of this post. Like scene handling, and editor gizmos, we will handle these when they become a necessity.
For this simple scene they shouldn't be, even though collectively this will be about a million triangles on the screen.

The three models on display are somewhat legendary in the graphics community. Suzanne is the monkey, built into [Blender](https://www.blender.org/).
The teapot is the [2025 - Cem Yuksel](https://graphics.cs.utah.edu/teapot/) version of the Utah Teapot, which has been around in some
for or another since 1975. The dragon is a 3D scan from the [The Stanford 3D Scanning Repository](https://graphics.stanford.edu/data/3Dscanrep/).

Each mesh is imported multiple times for demonstrating the difference between low/high triangle counts and flat/smooth shading.
Looking down the center and going right:
Suzanne: flat shading 967 triangles, smooth shading 967 triangles
Teapot: flat shading 7k triangles, smooth shading 7k triangles, flat shading 116k triangles
Dragon: flat shading 17k triangles, smooth shading 17k triangles, smooth shading 700k triangles

Additionally Suzanne is included twice more to the left with various normals flipped on the mesh for introducing artifacts as well. Both of these
models have 967 triangles, one is smooth the other flat shaded. These erroneous normals make them appear incorrect in the reference scene.

### Precursor to a debug shader

As the feature set of the engine grows, it is increasingly more important to have sufficient debugging capabilities. One of them is
a simple shader that can color the object based on its normals orientation. How this should work, and not be entirely boring or annoying is
unclear. In Blender and Godot they simply shade the front face of the triangle gray and the backface a darker gray.
To me this doesn't necessarily highlight any potential issues well enough.
What would be ideal is to color each triangle with some pleasing tones, that are simple enough to read, but aren't annoying. Furthermore
we should be able to tell for each face if all its normals are pointing in the same way or not, by highlighting this difference.
The human perception is most reactive to the red, green and blue color triplet. Red is a bit too harsh, so blue and green will be the base colors.
Red will be mixed in to highlight other inconsistencies.
Lastly the shader should track the camera and the angle it is looking at the object and shade appropriately. Hopefully this will lead to some,
relatively pleasing results that also highlight any errors with the meshes.

As a first step though, we will do something simpler. Put a simple directional light in world space. Color the faces bluer the closer their normals
align with the inverse of the light direction, and bluer otherwise.
This way as a start we don't have to track the camera, the "light" always stays in the same place allowing us to observe the coloring scheme from all
angles. In the next chapter then we can finish the desired implementation.

```gdshader
shader_type spatial;

// turn on wireframe by adding "wireframe"
render_mode unshaded, skip_vertex_transform;

const vec3 world_light_direction = normalize(vec3(1.0, -1.0, -1.0));

// flat means not to interpolate the value for the fragment shader
varying flat vec3 light_direction;

void vertex() {
	vec4 model_vertex_position = vec4(VERTEX, 1.0);
	NORMAL = MODELVIEW_NORMAL_MATRIX * NORMAL;
	VERTEX = (PROJECTION_MATRIX * MODELVIEW_MATRIX * model_vertex_position).xyz;
	// skip_vertex_transform disables Model and View transformations for a vertex, but
	// by setting POSITION ourselves even projection is disabled.
	// This is not necessary, but as a learning experience it is helpful to see the whole transformation
	// pipeline applied to each vertex, without it being obfuscated by some internal behavior.
	POSITION = PROJECTION_MATRIX * MODELVIEW_MATRIX * model_vertex_position;
	// The light direction only needs to transformed by the View matrix and since it it only a direction
	// only rotation is applied to it.
	light_direction = normalize((VIEW_MATRIX * vec4(world_light_direction, 0.0)).xyz);
}

void fragment() {
	vec3 normal = normalize(NORMAL);
	
	float similarity = (dot(-light_direction, normal) + 1.0) / 2.0;
	vec3 color = mix(vec3(0.0, 1.0, 0.0), vec3(0.0, 0.0, 1.0), similarity);
	// We are using linear values to define the green and blue colors and Godot
	// renders using linear color space, so no transformation is needed.
	ALBEDO = color;
}
```

## Importing the first .obj

As decided before the [Wavefront OBJ](https://paulbourke.net/dataformats/obj/) will be used as the file format of choice. The format is ancient and
many extensions have been added to it throughout the years. This complexity and the lack or original specifications, reference implementations makes
it error prone in practice. The issue can be sidestepped though, if we only focus on its most basic features and ignore all the rest.
In this case we only need it to be able to describe a **triangulated mesh** using positional/normal/uv vectors for each vertex.
That is it. Everything else we can ignore. With these restrictions a '.obj' file looks like this for a simple plane:

```text
# Blender 5.1.1
# www.blender.org
o Plane
v -1.000000 0.000000 1.000000
v 1.000000 0.000000 1.000000
v -1.000000 0.000000 -1.000000
v 1.000000 0.000000 -1.000000
vn -0.0000 1.0000 -0.0000
vt 1.000000 0.000000
vt 0.000000 1.000000
vt 0.000000 0.000000
vt 1.000000 1.000000
s 0
f 2/1/1 3/2/1 1/3/1
f 2/1/1 4/4/1 3/2/1
```

Components:
- '#' comment
- 'o' group name
- 'v' positional vector
- 'vn' normal vector
- 'vt' uv coordinate vector
- 'f' faces
- 's' smoothing group (we can ignore this)

That is all. First all the data is specified and put into their respective buffers, then the faces index these buffers using the format **vertex index/uv coordinate index/normal index**.
For some reason the indexes are counted from 1 instead of 0, but apart from that, this is it. (We ignore here of course that some vectors have optional components, that faces don't have
to specify everything only the position, that you can use negative indexes, that it isn't strictly specified that every vertex has to be defined at the beginning etc..)

When we implement the parser and import **Suzanne**, we get this

{{<figure src="suzanne_first_import.png" title="Suzanne">}}

What we see is **Suzanne** textured as the plane. This is not what we want but it is good demonstration of another problem. That of complexity. To render the plane, we needed
buffers to write the vertex positions/normals/texture coordinates into. Another buffer to index these buffers. A texture, that we had to mipmap, for which we had to allocate
another buffer. Then to access that texture we needed a sampler. A similar problem for rendering the cube, which also uses a different kind of texturing pattern.
The skybox is completely different as well. Now we want to render without textures by using some simple shaders instead. Managing all this is rather difficult. More difficult
it will get still when later on we want to actually add materials, lights, shadows etc.. Based on the combinations of the required features for rendering a given object
different pipelines, shaders and configurations must be set up.

In some engines this means that they have billions of shader pipelines. All tailored for a given permutation of features. In others they have a few "ubershaders" which handle
most of the possible configurations. In the earlier days the first approach was more prevalent, because the GPU was a very limited resource, it didn't like branches too much and the
register pressure was too high. To squeeze out all the performance it was more economical to generate custom setups for each task (the graphics weren't on today's complexity we might add).
This way one could avoid writing branches, as it was known exactly what will happen, before the code was even written.
Since then the GPUs have evolved a bit and are much better at branching. The graphics pipelines also got much more complex, to the point that the aforementioned possibly billions of
combinations are more of a detriment than a handy optimization. With so many possibilities and switching between different executions it is extremely difficult to retain any decent
cache coherency. This is truly a huge engineering problem and it is already causing problems for me. Never the less we must soldier on for a good while longer, while incrementally
refactoring sections if they become very troublesome.

### Implementing the "normal debug" shader precursor

After refactoring the main scene, separating out the plane and cube as the textured objects and building out a whole new pipeline for the "debug shader", finally Suzanne should
be shaded properly!

{{< youtube fZZsfGmxSR0 >}}

Oh no, that isn't quite right. When the camera orbits Suzanne blue sections turn green and green sections turn blue. How can this be? Deliberately restricted the implementation of
the shader, to always produce the same result regardless of the cameras or the objects orientation.

Here are the relevant parts of the shader:

```wgsl
const world_light_direction = normalize(vec3(1.0, -1.0, -1.0));

@vertex
fn vs_main(vertex: Vertex) -> VSOutput {
    var vsOut: VSOutput;

    // Compute the vertex position in device coordinates
    vsOut.position = global.view_projection * entity.world * vertex.position;

    // Orient the normals in world space
    vsOut.normal = entity.normal * vertex.normal;
    vsOut.light_direction = normalize((global.view * vec4f(world_light_direction, 0.0)).xyz);
    // the returned vector will automatically be normalized using w
    // [x,y,z,w] => [x/w, y/w, z/w, 1]
    return vsOut;
}

@fragment
fn fs_main(vsOut: VSOutput) -> @location(0) vec4<f32> {
    // All inter-stage variables get interpolated, so they
    // have to be renormalized if necessary.
    let normal = normalize(vsOut.normal);
    let light_direction = vsOut.light_direction;

    let similarity = (dot(-light_direction, normal) + 1.0) / 2.0;

    return vec4(mix(vec3f(0.0, 1.0, 0.0), vec3f(0.0, 0.0, 1.0), similarity), 1.0);
}
```

We can see that it is nearly exactly the same as the Godot reference implementation. A few minor differences are observable, for example
here a "view_projection * world" matrix is used and in Godot a "projection * modelview" combination. These are the exact same thing and will result
in the combined "model view projection" matrix. So the code is the same, yet still behaves differently.
Hmm, what can it be. After some experimentation it turned out that if the "light_direction" is simply set as the "world_light_direction", without
any transformations, the result will stabilize and appear as expected. Curious, curious, because that light was defined to be in "world space",
so it must be transformed by the "View" matrix. The transformation must be applied to it, otherwise it would not be oriented in the appropriate direction.
This can be confirmed, by disabling the transformation in the Godot reference. What we will see will be the exact same issue we observe here.
By the movement of the camera the shader output changes.
So in Godot, the transformation is necessary, while for us it is actively causing the issue? Marvelous.
Maybe there is a bug in the "normal matrix"? How could there be? In [Part 1](../part_1), the normal matrix already functioned
correctly with the camera orbiting around the object. But something is clearly not rotating when it should and bamm it hit me!
In my normal matrix does not contain the "view matrix", so it does not correct for the movement of the camera!
If we look at the Godot version, the matrix is called `MODELVIEW_NORMAL_MATRIX`. It contains the "model" and the "view" transformations!
Mine only contained the "model"! After fixing the issue the shader behaves as expected.

Okay, but then why was it working in [Part 1](../part_1)? Very simple, there the light, camera, vertex positions and the respective directions were
all calculated in "world" space (just the "model" matrix was applied). In this shader they were calculated in "model view" space.

## All objects ready, now course correcting

{{< figure src="scene.png" title="Imported objects">}}

All the target objects are imported and look as expected. After the **display transfer function** the colors too appear the same.

{{< figure src="voxon_vs_godot.png" title="Voxon vs Godot">}}

Let us see what happened with the framerates after adding this circa million polygons to the scene. As a reminder previously

`develop` build
```text
Avg. FPS: 2547.60
1% low: 1929.50
0.1% low: 1263.82
```

`release` build
```text
Avg. FPS: 10790.21
1% low: 6418.93
0.1% low: 2536.53
```

and now

`develop` build
```text
Avg. FPS: 1949.03
1% low: 1849.82
0.1% low: 1816.83
```

`release` build
```text
Avg. FPS: 4476.02
1% low: 1883.10
0.1% low: 1414.46
```

That is a rather big hit. In the `debug` build we have lost about **~600** frames per average, in the `release`
almost **~6000**. Nearly half the frames are gone with the wind after adding only this many polygons. To be honest,
not sure how many are too many, but nowadays a million, especially on the GPU sounds like an afterthought.
This should be investigated a bit further, before additional complications are added to the engine. For once, the code wasn't
kept too clean, secondly there are a few optimization ideas that should be put to the test.
Thus, instead of adding a wireframe rendering option, first we will break the engine and see how we can fix it.

## To the limits

Currently the code was written in the simplest copy/paste way possible. Completely disregarding any performance considerations.
Every mesh

Now that we have some meshes, which we can duplicate and position as required, the easiest way to increase the load on the GPU
is to just simply render more meshes. By adding the same mesh over and over, using the same shader a specific setup is reached.
One where most objects in the scene use the same mesh and the same shader. Which means that two optimization strategies can be
tested one after another.

Adding the smooth shaded utah teapot with about 7k polygons, **`16x16x16` = 4096** times, using up about **28'672'000** polygons, we still
hover around **90 FPS** on average in `release` build.

```text
Avg. FPS: 89.07
1% low: 38.90
0.1% low: 35.40
```

{{< figure src="utah_cube.png" title="Utah cube" >}}

In fact adding more of them after about `16x16x8` didn't seem to affect the FPS any more, even though we have no LOD or any type of
optimization on our side to speak off.
Maybe the vertex concentration is simply not high enough?

Replacing with the teapot cube with the smooth shaded stanford dragon using 700k polygons in a **`5*5*3` = 75** pattern, using up
**52'500'000** polygons seems to do the trick. Now in release build we still get about a 100 FPS on average but the 1% lows, especially
the 0.1% lows start to tank.

```text
Avg. FPS: 113.56
1% low: 27.00
0.1% low: 7.55
```

{{< figure src="dragon_cubish.png" title="Dragon cubelike">}}

This is getting noticeably choppier. More over, adding just one more layer on the `Y` axis makes it completely unusable.
A good candidate for observing how basic code restructuring and optimizations may affect performance.

At the moment every single object has it's own mesh instance in memory, its own buffers allocated on the GPU.
Within each render pipeline, there is a draw call for each individual object.

The stanford dragon is a very good example for the potential costs. Each 700k polygon stanford dragon uses up about
**66'844'704 bytes** of memory. Having 75 instances of them totals in **5'013'352'800 bytes** or **5 GB** of VRAM used.
CPU utilization barely reaches 1% which is a bit surprising, because every render cycle all objects get their
transform matrices recalculated and we have 87 objects to render. Admittedly, not that many, but still with a full
running operating system and calculating multiple 4x4 matrix multiplications for 87 objects, even at 4000 times a second
barely registers a 2 % load.
On the other hand GPU utilization is obviously at 100% all the times. We render as many frames as it is willing, and
as we see it, the CPU is not the bottleneck, so at least in this examination, just focusing on our handy dandy
framerate calculator we can observe how helpful each optimization is.

The first thing we can do is instancing. This is where we tell the GPU to draw.... TODO

After replacing all 700k polygon stanford dragons in the scene with instanced rendering we get the following:

`debug`
```text
Avg. FPS: 211.86
1% low: 89.69
0.1% low: 70.92
```

`release`
```text
Avg. FPS: 211.86
1% low: 89.69
0.1% low: 70.92
```

VRAM usage is substantially reduced, and performance became much more stable. Average FPS is good to be high, but more
important is that 1% and 0.1% lows don't drop now below 60. That means no hitching at all.

On the other hand if we simply replace all 700k poly stanford dragons with the 17k version ones we would get:

`debug`
```text
Avg. FPS: 1262.98
1% low: 1214.46
0.1% low: 1202.26
```

`relese`
```text
Avg. FPS: 5417.23
1% low: 1818.27
0.1% low: 1775.12
```

Replacing **52'500'000** polygons by only **75 * 17'000 =  1'250'000**, a 42x reduction,
resulted in a worst case performance gain of 25x.

Just out of curiosity how does Godot perform? Using Vulkan 1.4.335, without the 75 extra dragons
the editor reports **~2000 FPS**. If all 75 dragons are present it still manages to crank out **~670 FPS**. If on the
other hand we completely turn of **Mesh LOD** optimization, the framerate drops to about **125** FPS.
Now this is a much fairer comparison to **Voxon**, as we don't have Mesh LOD at all.
When the project is built, with all its optimizations in debug/release it will run at about **2200/2600 FPS**.
Pretty impressive to be honest. Didn't expect it to be so performant.

The lesson seems to be very simple. More draw calls is bad. More polygons, even worse.

## Wireframes

Rendering a wireframe representation of a given mesh is both simple and not.

### Using barycentric coordinates

Firstly we could do it using [barycentric coordinates](https://en.wikipedia.org/wiki/Barycentric_coordinate_system).
This would lead to a very simple implementation where the `vertex shader` generates a vector for each vertex based on its index.
This vector, the barycentric coordinate will be calculated like so:

```
vsOut.barycentric_coordinate = vec3f(0);
vsOut.barycentric_coordinate[vertex.vertex_index % 3] = 1.0;
```

In other words each vertex will become one of `<1.0, 0.0, 0.0>, <0.0, 1.0, 0.0>, <0.0, 0.0, 1.0>`, which is analogous to the
unit weights assigned to each vertex in the barycentric coordinate system. Then due to the linear interpolation, by the
pixel/fragment shader stage we would have the actual barycentric coordinate calculated for each fragment for basically free.
Then finally we could pick the smallest weight (the smallest distance to an edge), check if it is above or below a threshold
and set the transparency of the fragment to 0.0 or 1.0. Above the threshold it should be fully transparent, below, fully opaque.

{{< figure src="barycentric_wireframe_no_culling.png" title="Barycentric wireframe">}}

There is a problem though. Chiefly that we can't define the width of the lines in pixels or meters. Nor are they of the same
width using the same threshold value, in this case 0.03.
To see a bit easier, the same wireframe, but only the front faces are visible:

{{< figure src="barycentric_wireframe_culled.png" title="Front faces Barycentric wireframe">}}

Take a look at triangle `A` and `B`. Observe that `A` has wider lines than `B`. The reason is simple. Barycentric coordinates
don't take account for the size of the triangle in screen space. Thus a triangle that takes up more screen space will have
proportionally wider lines than a smaller one. The effect is initially subtle, but becomes rather annoying after observing it.

There are sources out there where they apply additional transformations to the barycentric coordinates within the fragment shader and
calculate a so called edge factor:

```text
fn edge_factor(barycentric_coordinate: vec3f) -> f32 {
    let d = fwidth(barycentric_coordinate);
    let aliased = smoothstep(vec3f(0.0), d * line_width, barycentric_coordinate);
    return min(min(aliased.x, aliased.y), aliased.z);
}
```

First of all what are we doing?
[fwidth](https://webgpufundamentals.org/webgpu/lessons/webgpu-wgsl-function-reference.html#func-fwidth) will calculate the partial derivative 
of the vertex coordinates in regards to the x and y window coordinates. Quite frankly, not sure what that exactly means and there
is precious little documentation about it. The [smoothstep](https://webgpufundamentals.org/webgpu/lessons/webgpu-wgsl-function-reference.html#func-smoothstep)
is simply interpolates between two values in a [non-linear fashion](https://en.wikipedia.org/wiki/Smoothstep). This in practice means
that there won't be a hard transition between the two values, but a soft one, in other words, antialiasing is applied.
Last line is just as before, get the closest edge 'distance'.
This doesn't work either. It alleviates the scaling problem to some extent but it doesn't solve it. Why?
We are still working within a barycentric system which is disconnected from the screen space. `line_width` is in incomprehensible units, because
all the other values are in incomprehensible units as well. To demonstrate why this version doesn't work either, here is a comparison between it
and how the same scene would look in Godot.

{{< figure src="fwidth_bary_comparison.png" title="Wireframe comparison" >}}

On the left our implementation using `edge_factor` to generate antialiased wireframe lines, on the left Godot.
Notice how in Godot, every line is exactly one pixel wide, regardless it's distance from the camera. While on the left lines even
close to the camera have differing widths, which is not just the result of the antialiasing step. Worse, lines further away will appear
even wider. The effect is even more jarring if we navigate the 3D space.

A better approach is needed. One where line widths can be set according to some human compatible metric and aren't affected
by the size of each triangle or by projection.

### Calculating distance to nearest edge
