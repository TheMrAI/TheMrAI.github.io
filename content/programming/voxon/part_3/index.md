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
For some reason the indexes are counted from 1 instead of 0, but apart from that this is it. (We ignore here of course that some vectors have optional components, that faces don't have
to specify only the position, etc..)

When we implement the parser and import **Suzanne**, we get this

{{<figure src="suzanne_first_import.png">}}

What we see is **Suzanne** textured as the plane. This is not what we want but it is good demonstrating another problem. That of complexity. To render the plane, we needed
buffers to write the vertex positions/normals/texture coordinates into. Another buffer to index this buffer. A texture, that we had to mipmap, for which we had to allocate
another buffer. Then to access that texture we needed a sampler. A similar problem for rendering the cube, which also uses a different kind of texturing pattern.
The skybox is completely different as well. Now we want to render without textures by using some simple shaders instead. Managing all this is rather difficult. More difficult
it will get still, when later on we want to actually add materials, lights, shadows etc.. Based on the combinations of the required features for rendering a given object
different pipelines, shaders and configurations must be set up.
In some engines this means that they have billions of shader pipelines. All tailored for a given permutation of features. In others they have a few "ubershaders" which handle
most of the possible configurations. In the earlier days the first approach was more prevalent, because the GPU was a very limited resource, it didn't like branches too much and the
register pressure was too high. To squeeze out all the performance it was more economical to generate custom setups for each task (the graphics weren't on today's complexity we might add).
This way one could avoid writing branches, as it was known exactly what will happen, before the code was even written.
Since then the GPUs have evolved a bit and are much better at branching. The graphics pipelines also got much more complex, to the point that the aformentioned possibly billions of
combinations are more of a detriment than a handy optimization. With so many possibilities and switching between different executions it is extremely difficult if not completely
impossible to retain any type of cache coherency. When that is gone, performance can drop thousand folds. Hence ubershaders were born.
