---
layout: page
title: Make it prettier
date: "2026-03-16"
summary: "Rework the scene with simple textures"
draft: true 
---

In the previous post a lot of things have happened and went sideways, making
the whole process a lot slower than it needed to be. Acceptable outcome, since
many moving parts had to work before something could actually happen.

This time around we are a bit better off and can afford to plan a bit ahead and test things
out with Godot beforehand.

The goal is to make the overall scene a bit more presentable. While originally I wanted to
proceed with light and shadow implementations, eventually decided against it. The reason is
that there is a more fundamental concept in graphics that is much more valuable.

## Textures

Textures are a simple concept, brilliantly utilized in multiple ways. A chunk of memory
organized into an array (1D texture), a matrix (2D texture) or 3D.
A texture can be used to wrap an image around an object, define normals for the object,
motion vectors, material properties and so on.
They are fundamental for shadow algorithms, light baking and complex material properties.

To demonstrate how massively they can improve the visual experience observe the below comparison.

{{<figure src="../part_1/godot_reproduction.png" title="Previous state">}}
{{<figure src="few_textures.png" title="Just a few textures">}}

Before we had a light which produced a green .. err .. glow I suppose. Now, the light
has been removed and a texture was added to the ground plane, the cube and the sky.
The absence of shadows makes the image a bit unnerving, but this is all we will need.

The only other thing we needed again was a simple Godot [spatial shader](https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/spatial_shader.html#doc-spatial-shader)
which circumvented basically all of Godot's shading and lightning system ensuring that the only thing
used for "coloring" a pixel was just a texture.

```gdshader
shader_type spatial;

render_mode unshaded;

uniform sampler2D plane_texture;
uniform vec2 uv_scale;

void vertex() {
	UV = UV * uv_scale;
}

void fragment() {
	vec3 sample = texture(plane_texture, UV).rgb;
	// force SRGB color
	ALBEDO = mix(pow((sample + vec3(0.055)) * (1.0 / (1.0 + 0.055)), vec3(2.4)), sample * (1.0 / 12.92), lessThan(sample,vec3(0.04045)));
}
```

The only "hack" that needed to be added was forcing the sRGB color space. Not entirely sure what that piece of code does, less so why, but seems to produce
the expected result. Without it, the textures will not appear as they should and would have this washed out appearance to them.

{{<figure src="few_textures_without_srgb.png" title="Without forced sRGB">}}

This way, hopefully, we can relatively easily reproduce the results.

### Sources

All textures are open source under the [CCO 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/) license.
You can use them for free, for commercial products as well, without any restrictions.
Attributions are not required, but it is the least we can do to the artists/original sources.

The skybox is [Kloofendal 48d Partly Cloudy (Pure Sky)](https://polyhaven.com/a/kloofendal_48d_partly_cloudy_puresky), from
[Poly Haven](https://polyhaven.com/). Extremely high quality textures, for completely free!

The ground texture is taken from the [Prototype Textures](https://kenney-assets.itch.io/prototype-textures) pack.

The cube texture was made by hand, but you may use it under the [CCO 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/)
license as well.

{{<figure src="cube_atlas.png" title="Cube texture">}}

The following posts were invaluable in understanding what the basic concepts were and how one should think about them:
- [WebGPU Textures](https://webgpufundamentals.org/webgpu/lessons/webgpu-textures.html)
- [WebGPU Loading Images into Textures](https://webgpufundamentals.org/webgpu/lessons/webgpu-importing-textures.html)
- [WebGPU Cubemaps](https://webgpufundamentals.org/webgpu/lessons/webgpu-cube-maps.html)
- [iWebGPU Environment Maps](https://webgpufundamentals.org/webgpu/lessons/webgpu-environment-maps.html)

We will go through the same topics and concepts, but only limited to the extent that it was necessary for duplicating
the target scene.

## The process

First, the light and any related code has been removed, and the fragment shader now simply returns a 20% grey color for
any pixel. The plane was scaled to be 100x100 meters big and with this, the basic setup of our example scene has been achieved.

{{<figure src="stripped_scene.png" title="Scene">}}

Next a simplest 2x2 texture was made to be used for the plane of the format:
```
green, blue
blue, green
```

This texture is repeated 50 times for the plane, resulting in a checkerboard pattern, where each rectangle is
exactly a meter wide/tall. Observe below how the 2x2 cube is exactly a green and blue rectangle wide.

{{<figure src="checkerboarded.png" title="Checkerboard">}}

These scales are very important, especially in such barren scenes, as they act as a reference for scale and orientation.

One thing that surprised me is that a **texture** and a **sampler** are special resources which cannot be handled as simple uniforms
with WebGPU.

Initially, this was what I wanted to do:

```wgsl
struct Entity {
    world: mat4x4f,
    normal: mat3x3f,
    texture_sampler: sampler,
    texture: texture_2d<f32>,
}

@group(1)
@binding(0)
var<uniform> entity: Entity;
```

This though is simply not possible, because a **texture** and the **sampler** is a different type of resource, and even though
it doesn't change for a draw call (hence I thought it could be a uniform), it must be bound separately.

```wgsl
struct Entity {
    world: mat4x4f,
    normal: mat3x3f,
}

@group(1)
@binding(0)
var<uniform> entity: Entity;

@group(2)
@binding(0)
var texture_sampler: sampler;

@group(2)
@binding(1)
var texture: texture_2d<f32>;
```

Why exactly this may be is not clear. Perhaps because there are specialized internal representations and/or specific hardware elements
which the GPU handles? A texture sampler is a built-in hardware component as it has to be extremely fast, similarly a texture
is stored in GPU memory in a way that accelerates its access.

For example a simple 4x4 matrix like this:

```text
i\j 0 1 2 3 
0   a b c d
1   e f g h
2   i j k l
3   m n o p
```

In the most common, row major sequential memory layout format, it would look like this:

```text
a b c d
e f g h
i j k l
m n o p
```

For demonstration purposes a line break was inserted at the end of each row. In practice this would
just be a continuous piece of memory.

```text
a b c d e f g h i j k l m n o p
```

This layout is very good for iterating the rows of the matrix very fast, because by loading a line into
memory, a number of following elements will be loaded into the cache as well.

Textures though aren't accessed in a sequential manner. A **sampler** has to be able to access the neighboring
**texels** at a given coordinate very fast. A **texel** is a pixel of a texture.
Instead of a sequential memory layout a [Z-order curve](https://en.wikipedia.org/wiki/Z-order_curve) is used.

Using Z-order curve memory layout the above matrix would look like this in memory:

```text
a b e f c d g h i j m n k l o p
```

More on GPU cache locality can be found at [Gpu cache hierarchy](https://charlesgrassi.dev/blog/gpu-cache-hierarchy/).

### Loading the plane texture

You would think that just loading a [PNG](https://en.wikipedia.org/wiki/PNG) should be easy right?
It is a just a simple file format that has been around forever. Tried loading the texture:

{{< figure src="texture_01.png" title="Plane texture">}}

with the [png](https://docs.rs/png/0.18.1/png/index.html) rust library.
It loaded, but something didn't seem right, WebGPU would not accept it. 
Tried loading the wgpu logo, following the [wgpu bunnymark example](https://github.com/gfx-rs/wgpu/blob/0ca8ba04acbca7caa964d64492da0e831d9da35c/examples/features/src/bunnymark/mod.rs#L241)
and that worked.

{{< figure src="logo.png" title="Wgpu logo">}}

If I were to open the two textures with **Krita** it would say they have the format: **RGB/Alpha (8-bit integer/channel)**, using the
**sRGB-elle-V2-srgbtrc.icc** profile. On the other hand, when they were to be opened with [Zed](https://zed.dev/),
for the **texture_01.png** it would be **3 channels 24 bits per pixel**, for the **logo.png** **4 channels, 32 bits per pixel**.
It seems the alpha channel is missing from **texture_01.png**. WebGPU has no support for textures with ['rgb' format](https://www.w3.org/TR/webgpu/#texture-formats-tier1) where
each channel takes up 8 bits only, the closest is "rgba8unorm" or "rgba-8unorm-srgb". This would not be a problem, as it should
not be too big of an issue to add an alpha channel for every pixel with the maximum value of 255 signaling that it should be
fully opaque. Another problem arises! **texture_01.png** does not store the pixels as RGB color values, it stores them in a format called [Indexed](https://www.w3.org/TR/2003/REC-PNG-20031110/#3Abbreviations).
This simply means that somewhere within the png, a section called a **pallet** is allocated which stores all the distinct pixel colors within the image,
and the pixels just index into this array.
This still could have, been okay, still could have produced the appropriate format, but the data [png](https://docs.rs/png/0.18.1/png/index.html)
provided for **texture_01.png** made no sense.

```text
Info { width: 1024, height: 1024, color_type: Indexed, ..., palette: Some([89, 89, 91, 107, 107, 109, 91, 91, 93, 71, 71, 73, 255, 255, 255, 51, 51, 53]), ... }
Buffer: [68, 68, 68, 68, 68, 68, ... ]
```

Where we see the palette contains 6 distinct 3*8 bit colors, confirming the "24 bits per pixel" reported by Zed (not sure Krita meant though). What doesn't make sense is that the buffer contains values that don't look
like indexes. And this is the point where the investigation stops. Not sure if the png library is buggy or not, the issue can be sidestepped by loading the image into Krita and saving it as
a PNG with the default settings (Force convert to sRGB, Store alpha channel (transparency)). This way its format will be exactly what we need **RGBA 8 bit per channel**.

To us this is only interesting if you want to do it by hand. Didn't encounter any issues with the textures while constructing the reference scene in Godot, so there is nothing wrong with them,
but it seems to be very important to know what exactly is in a PNG, before one can use it to directly communicate with the GPU.

For all our troubles we get the scene:

{{< figure src="plane_textured_noisy.png" title="Uhh">}}

This doesn't look quite right. The high frequency information from the texture is lost. One thing we need to change is use a `linear` blending, but
that won't solve our problems either.

{{< figure src="plane_textured_linear.png" title="Linear interpolation">}}

As it can be seen the lines became smoother, but the texturing still falls apart completely after a certain distance from the camera.
The issues is, as with many things in graphics, the discrepancy between the sampling frequency and the frequency of the data within the texture
we are sampling. In our case the texture is mostly high frequency data, in layman terms lines or hard/crisp edges.
The further away the texture, the more unlikely is, that our low frequency sampling will be able to gather the requisite information.
The result is that seemingly randomly the lines in the texture disappear.

[todo sampling demonstration example]

The sampling problem, lays within the domain of digital signal processing, about which whole books have been written about. However, in computer
graphics tricks and shortcuts are more important if they are performant and get us 90% the way there than the actually correct solution.
And it so happens there is a rather simple solution to this issue. It is called [mimpmaps](https://en.wikipedia.org/wiki/Mipmap).
The idea is staggeringly simple. We don't want to increase the sampling rate, because that is costly. Instead we can generate smaller
and smaller textures, constantly smearing the image a bit more and more. Then the further the to be textured point is from the camera the more
smeared texture we use.
In practice this means that high frequency information slowly turns into low frequency information, which can be sampled well with a low frequency.

{{< figure src="plane_textured_mipmap.png" title="Mipmapping">}}

If we look at the line starting from the bottom center of the screen, we can observe that as it gets further away from us, it gets wider and blurrier.
Not ideal, but it is cheap to compute, and in practice it sells the illusion very well.

### UV coordinates

https://www.lightmap.co.uk/blog/whatisanhdrimap/
https://www.cgibackgrounds.com/blog/what-is-an-hdri


## The cube texture

If you have heard about textures before you have probably heard about an UV maps/UV wrapping. Most likely you have seen one for a cube
and now you may be wondering why does the one above look like it does.

[[todo cube map demo]]

Because that is not how it is done in practice as it is very wasteful. It can work of course, but just observe how much of the image is wasted
for nothing at all.
The proper way of doing it is by **texture atlases**. Don't be alarmed, a texture atlas is nothing but a texture, into which other smaller textures
have been aggregated into.

{{<figure src="cube_atlas.png" title="Cube texture">}}

When we map it onto our cube we get:

{{<figure src="cube_textured.png" title="Textured cube">}}

Which is good, but unfortunately not perfect. As described before mipmaps are necessary for any half decent texture sampling, but this also causes
an issue.

{{< figure src="cube_mipmap_error.png" title="Atlas mipmap error" >}}

Because all the textures are merged into one atlas, when the mipmaps are generated they will get mixed together along the boundaries, not to mention
the sample on the edges will sample into the neighbours as well. Which can lead to all kind of unwanted artifacts like the one above. The red pixels
are blended into the edges of the blue and green sides.

There are ways of mitigating this, for example described in [Texture atlases, wrapping and mip mapping](https://0fps.net/2013/07/09/texture-atlases-wrapping-and-mip-mapping/),
but for the time being such optimizations will be omitted. Our goal for now is to have the same scene as in the Godot example, which exhibits the exact
same behaviour.

## The Skybox

First it is best to clean up some definitions. Especially if one is new to these technologies it is very confusing how they are mixed up left and right.

### HDR, HDRI definitions

[Kloofendal 48d Partly Cloudy (Pure Sky)](https://polyhaven.com/a/kloofendal_48d_partly_cloudy_puresky) is an **image** created using **HDRI (High dynamic
range imaging)**. It uses **HDR (High dynamic range)** to more naturally capture the wildly differing luminosity values in a scene.
It is not HDR, it uses HDR (high dynamic range) instead of SDR (standard dynamic range).
It is not an HDRI image. HDRI is the technology/methotology for capturing it
([How to Create High Quality HDR Environments](https://blog.polyhaven.com/how-to-create-high-quality-hdri/) if you want to read more). There is no correct way to
referring to their type, in leu of a proper one, often their extension is used which is either **.HDR** or **.EXR**, or this mouthful appears
**HDRI image** (high dynamic range imaging image). The situation is a tad bit unfortunate, but it is what it is.

Another misnomer is that Skybox-es are often just equated with an HDRI image. Skyboxes often use HDRI images as textures, but any other object could use
an HDRI image as well. It is simply more prevalent that the skybox is an HDRI image, because that alone can help light the scene appropriately.

### Skybox implementation

There are two ways of implementing a skybox I have found so far.

Either using cubemaps or a single image using equirectangular projection.
They are rather similar in their basic concept.
Take the camera, project a number of rays from the eye through the center of each pixel on the **Z near**-plane,
find the direction of these rays, then sample the texture that has been conveniently placed in model space using these
directions.

Here is where we can again be grateful for the absolute brilliance of the people coming before us and marvel at the elegance by which this issue
has been solved. In practice we don't project any rays at all, but the result will be exactly the same. 

#### Equirectangular projection

Our chosen skybox texture [Kloofendal 48d Partly Cloudy (Pure Sky)](https://polyhaven.com/a/kloofendal_48d_partly_cloudy_puresky) is an
HDRI image, produced using equirectangular projection. 

{{< figure src="sky_equirectangular.png" title="Equirectangular projection">}}

When first seeing such an image it is a bit difficult to understand what is that you are actually seeing, it looks a bit warped and just wrong.
Even though you have seen such projections before, if you have seen a map of Earth. Chances are, it was made using equirectangular projection.
The picture of Earth doesn't bother you, because while it is not correct, it is likely the only projection you have seen, so it appears correct
by default. In contrast this sky texture does not as the characteristical warping/streching, especially close to the top and bottom of the image,
makes it feel wrong. Such a projection is not really made for our minds, but if we were to wrap it back onto a sphere it would appear "correctly".

[Equrectangular projection](https://en.wikipedia.org/wiki/Equirectangular_projection) describes the math behind the transformation.
Conceptually it is even simpler. Imagine there is a unit sphere centered at origo, where a camera is located at as well.
This camera rotate clockwise on the Y axis (or an axis of you choosing really) and scan whatever it sees, through the points of the unit sphere
and write it to a 2D texture. During this scanning it always scan exactly Pi radians (180 degrees) vertically and 2*Pi radians (360 degrees)
horizontally. Producing an image which is 2*Pi wide and Pi tall. In other words twice as wide as it is tall, 2:1. (Apparently there are equirectangular
projections that only use half of the vertical resolution, resulting in a ratio of 4:1.)

[todo: demonstration of this scanning]

Such images seem to be a good candidate for a skymaps/environmental maps as they encode into a single texture the whole range of surrounding visuals.
This is why I would have preferred to use one as it is, but after the curvebals coming from just reading a *.PNG* image, it seemed to make more
sense not to try and battle the hdr file formats *.HDR* or *.EXR*. Not to mention the challenges that may arise from having to handle
the extended brightness ranges encoded in such an image.
For these reasons, it was decided that instead an older technology of cubemaps will be used.

#### Skybox using a cubemap

A cubemap is nothing really just a simple axis aligned cube, that has been textured according to the requirements of our
graphics API.
In our case WebGPU, defines the requirements on a cubemap texture like so:

{{< figure src="cubemap_faces.png" title="Cubemap faces" >}}

Which means that it needs exactly six textures of the same width and height in the order for the cube's sides. In
this case: +X, -X, +Y, -Y, +Z, -Z. That is a 2D array texture. This is not a 3D texture, even though we have a 3rd
dimension. These are separate textures, that can be simply sampled by a vector directly, due to the built in hardware support.

Using the tool [panorama-to-cubemap](https://greggman.github.io/panorama-to-cubemap/) we could turn our **EXR** equirectangular
skymap texture into a cubemap:

{{< figure src="sky_sdr_cubemap.png" title="Sky cubemap">}}

Notice that if you were to fold up the cube map, it would would look like and be oriented exactly the same as the cube in our
scene. +Z facing the camera, +X going right and +Y up. The right-handed Y up coordinate system, with the standard camera direction
looking down into the -Z direction.
