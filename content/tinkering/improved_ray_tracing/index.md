---
layout: page
title: Improving ray tracing speeds by ray marching
summary: Demonstrating the potential in voxelizing 3D space for ray marching, using a modified 3D DDA algorithm.
date: "2024-05-11"
---

## Inspiration

After finishing [Ray tracing]({{< ref "/tinkering/ray_tracing" >}}), I was left unsatisfied with the end result.
The original implementation had a number of shortcomings that I didn't like. The structuring of the code was confusing and
illogical, it didn't employ proper multithreading support. It didn't use meshes for representing the objects, which I have
felt were really constraining its capabilities.

This is why I have decided to completely rewrite it, according to what I have learned from my previous implementation. The goal
was to make it handle proper meshes, be as parallel as I can make it and run fully on the CPU.

After doing all that, I have realized something very important, which was expected but not to this extent. Collision detections
are exceptionally expensive when it comes to ray tracing. And when transitioning from a ray tracer which used the mathematical
representation for each shape, the number of collision checks for each ray bounce where exactly the number of objects in the scene.
However, after transitioning into using 3D meshes, with triangles, one will quickly realize that this naive approach will no longer work.
Every mesh, depending on its complexity will be constructed from a number of triangles. A complexity that quickly grows even for seemingly
simple objects. A simple sphere would turn into thousands of triangles, just so that it can look something like a sphere. Increasing the
number of possible collision checks from one object (if it is described as a mathematical formula), into thousands (if described as a mesh).
Even worse, this cannot be resolved by simple techniques, like bounding boxes. What if your ray travels inside a sphere? Then
the bounding box can't help.

All this turned a few week long project into months of research and experimentation. First I will show you the improvements that were
made, and then take a look at what was done.

## Measurements

A number of scenes were made, mostly for testing purposes and here I will list them all with their rendering speed using my original naive
approach and the optimized one.

All measurements were done on an [Intel Core i7-13700K](https://www.intel.com/content/www/us/en/products/sku/230500/intel-core-i713700k-processor-30m-cache-up-to-5-40-ghz/specifications.html), with
a total of 24 threads.

If you would like to try it yourself, for the original measurements commit [4d3ef310be0f1282cd747a4381b59f23903673b0](https://github.com/TheMrAI/ray_buster/commit/4d3ef310be0f1282cd747a4381b59f23903673b0)
was used, while for the improved [900afa5076456e5863c074761e49b2d1e1f8175b](https://github.com/TheMrAI/ray_buster/commit/900afa5076456e5863c074761e49b2d1e1f8175b).
All scenes use their default configurations, which remained unchanged between the two commits.

As it can be seen all simple scenes, where there are only a few planes/cuboids the improved algorithm produces a slightly worse result. On the other hand when mesh complexity starts to increase, as with
spheres, especially if they are metallic or dielectric the improved algorithm provides a huge improvement. This can be observed most clearly on the final scene of the floating-sphere.

### Test scenes

#### test-cuboid-emissive

{{< figure src="test-cuboid-emissive.jpg" >}}

**Original**

```text
real    0m0.302s
user    0m3.526s
sys     0m0.126s
```

**Improved**

```text
real    0m0.429s
user    0m5.914s
sys     0m0.068s
```

#### test-cuboid-inside-lambertian

{{< figure src="test-cuboid-inside-lambertian.jpg" >}}

**Original**

```text
real    0m0.451s
user    0m6.654s
sys     0m0.118s
```

**Improved**

```text
real    0m0.553s
user    0m9.608s
sys     0m0.097s
```

#### test-cuboid-inside-metal

{{< figure src="test-cuboid-inside-metal.jpg" >}}

**Original**

```text
real    0m1.134s
user    0m19.824s
sys     0m0.099s
```

**Improved**

```text
real    0m1.614s
user    0m30.278s
sys     0m0.119s
```

#### test-cuboid-material

{{< figure src="test-cuboid-material.jpg" >}}

**Original**

```text
real    0m0.588s
user    0m10.723s
sys     0m0.059s
```

**Improved**

```text
real    0m0.559s
user    0m7.901s
sys     0m0.158s
```

#### test-cuboid-rotate

{{< figure src="test-cuboid-rotate.jpg" >}}

**Original**

```text
real    0m0.728s
user    0m12.889s
sys     0m0.196s
```

**Improved**

```text
real    0m0.570s
user    0m8.875s
sys     0m0.078s
```

#### test-cuboid-scale

{{< figure src="test-cuboid-scale.jpg" >}}

**Original**

```text
real    0m0.750s
user    0m12.623s
sys     0m0.050s
```

**Improved**

```text
real    0m0.471s
user    0m8.626s
sys     0m0.050s
```

#### test-icosphere-emissive

{{< figure src="test-icosphere-emissive.jpg" >}}

**Original**

```text
real    0m1.306s
user    0m25.769s
sys     0m0.060s
```

**Improved**

```text
real    0m0.997s
user    0m16.960s
sys     0m0.140s
```

#### test-icosphere-inside-lambertian

{{< figure src="test-icosphere-inside-lambertian.jpg" >}}

**Original**

```text
real    0m1.248s
user    0m25.188s
sys     0m0.177s
```

**Improved**

```text
real    0m0.742s
user    0m13.453s
sys     0m0.059s
```

#### test-icosphere-inside-metal

{{< figure src="test-icosphere-inside-metal.jpg" >}}

**Original**

```text
real    2m13.962s
user    49m29.813s
sys     0m0.549s
```

**Improved**

```text
real    0m17.257s
user    6m10.467s
sys     0m0.180s
```

#### test-icosphere-material

{{< figure src="test-icosphere-material.jpg" >}}

**Original**

```text
real    0m17.503s
user    6m33.020s
sys     0m0.120s
```

**Improved**

```text
real    0m7.679s
user    2m51.226s
sys     0m0.130s
```

#### test-icosphere-rotate

{{< figure src="test-icosphere-rotate.jpg" >}}

**Original**

```text
real    0m4.515s
user    1m33.917s
sys     0m0.090s
```

**Improved**

```text
real    0m2.869s
user    0m57.176s
sys     0m0.149s
```

#### test-icosphere-scale

{{< figure src="test-icosphere-scale.jpg" >}}

**Original**

```text
real    0m6.186s
user    2m12.607s
sys     0m0.150s
```

**Improved**

```text
real    0m3.405s
user    1m12.792s
sys     0m0.139s
```

#### test-plane-emissive

{{< figure src="test-plane-emissive.jpg" >}}

**Original**

```text
real    0m1.297s
user    0m25.808s
sys     0m0.069s
```

**Improved**

```text
real    0m0.470s
user    0m6.901s
sys     0m0.040s
```

#### test-plane-material

{{< figure src="test-plane-material.jpg" >}}

**Original**

```text
real    0m0.175s
user    0m1.328s
sys     0m0.065s
```

**Improved**

```text
real    0m0.261s
user    0m2.473s
sys     0m0.097s
```

#### test-plane-material-collision-direction

{{< figure src="test-plane-material-collision-direction.jpg" >}}

**Original**

```text
real    0m0.280s
user    0m3.124s
sys     0m0.136s
```

**Improved**

```text
real    0m0.344s
user    0m4.083s
sys     0m0.134s
```

#### test-plane-rotate

{{< figure src="test-plane-rotate.jpg" >}}

**Original**

```text
real    0m0.174s
user    0m1.664s
sys     0m0.094s
```

**Improved**

```text
real    0m0.214s
user    0m2.620s
sys     0m0.066s
```

#### test-plane-scale

{{< figure src="test-plane-scale.jpg" >}}

**Original**

```text
real    0m0.187s
user    0m1.273s
sys     0m0.073s
```

**Improved**

```text
real    0m0.277s
user    0m2.427s
sys     0m0.077s
```

### Scenes

#### Cornell box

{{< figure src="cornell-box.jpg" >}}

**Original**

```text
real    0m15.764s
user    5m50.759s
sys     0m0.229s
```

**Improved**

```text
real    0m16.608s
user    6m9.349s
sys     0m0.269s
```

#### Cornell box advanced

{{< figure src="cornell-box-advanced.jpg" >}}

#### Original

```text
real    6m25.268s
user    145m28.717s
sys     0m1.629s
```

#### Improved

```text
real    1m22.157s
user    30m13.923s
sys     0m1.128s
```

#### Floating sphere

{{< figure src="floating-sphere.jpg" >}}

**Original**

```text
real    121m57.616s
user    2770m57.033s
sys     0m13.930s
```

**Improved**

```text
real    0m51.283s
user    18m24.359s
sys     0m0.879s
```

This is by far the biggest improvement. The scene was particularly hard on the naive method, because all of it took place inside a giant metallic icosphere, which alone
consists of thousands of triangles, not to mention the dielectric sphere at the center, which has even more of them.

## Improvements
