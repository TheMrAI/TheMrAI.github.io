---
layout: page
title: Auto pipelining
date: "2026-06-09"
summary: "Some refactoring and automatic pipelining"
draft:  true
---

Prior to this project I had close to zero experience (ehh call it an even zero) with game engine programming.
As a learning exercise a lot of mess was made. That is fine. Bumbling through the dark without knowing how to do
something inevitably leads to a mess. We learn from this mess and fix it in due time.

Had no misconceptions about game engines being simple in the slightest,
but nothing prepared me so far to the amount of sheer complexity and considerations that need to be made from the start.

Believed that it had to be similar to other engineering tasks. Even if you don't know what you are doing, small incremental
changes should lead you to a semi optimal outcome. As an exercise spent about 2 weeks just refactoring the code to make it
simpler in the hopes of observing some patterns that can be later used to build on. In some parts this was a success, in others
an abject failure. Let me show you why.

## ECM or ECS

There seems to be two major approaches in taming the complexity spirit demon residing in every
game engine. Furthermore it seems it doesn't matter which one you choose, suffering is guaranteed, just in
different ways.

First off, we have ECM - Entity component model, Entity system (ES), **Game-object component model**. Many names for the
same thing, confusing all initiates. This is basically the object oriented approach.
[Game Programming Patterns](https://gameprogrammingpatterns.com/) by **Robert Nystrom** is a very good read in this regard.
Presents the patterns in a fun, understandable manner. While it fails to show the big picture it demonstrates the pieces very well.
If you are as unfamiliar with the topics as I was, for this discussion it is recommended to read:
- [Game Loop](https://gameprogrammingpatterns.com/game-loop.html)
- [Component](https://gameprogrammingpatterns.com/component.html) <- the key to this pattern
- [Event Queue](https://gameprogrammingpatterns.com/event-queue.html)
- [Data Locality](https://gameprogrammingpatterns.com/data-locality.html)
- [Spatial Partition](https://gameprogrammingpatterns.com/spatial-partition.html)

It isn't necessary to fully understand them, but at least observe what type of problems have to be solved.

Second is ECS, which has only one name that I could stumble upon, **Entity component system**.
This is the data oriented approach. A lot of good information can be found here: 
[Entity Component System FAQ](https://github.com/SanderMertens/ecs-faq#how-is-ecs-different-from-entity-component-frameworks)

To me the most helpful ones were (all written by [Sander Mertens](https://ajmmertens.medium.com/)):
- [Building an ECS #1](https://ajmmertens.medium.com/building-an-ecs-1-where-are-my-entities-and-components-63d07c7da742)
- [Building an ECS #2](https://ajmmertens.medium.com/building-an-ecs-2-archetypes-and-vectorization-fe21690805f9)
- [Building an ECS #3](https://ajmmertens.medium.com/building-an-ecs-storage-in-pictures-642b8bfd6e04)
- [Why Vanilla ECS Is Not Enough](https://ajmmertens.medium.com/why-vanilla-ecs-is-not-enough-d7ed4e3bebe5)
- [Building Games in ECS with Entity Relationships](https://ajmmertens.medium.com/building-games-in-ecs-with-entity-relationships-657275ba2c6c)
- [Making the most of ECS identifiers](https://ajmmertens.medium.com/doing-a-lot-with-a-little-ecs-identifiers-25a72bd2647)
- [Why Storing State Machines in ECS is a bad idea](https://ajmmertens.medium.com/why-storing-state-machines-in-ecs-is-a-bad-idea-742de7a18e59)

Furthermore, [Events in Entity Component Systems](https://medium.com/@casual.evil52/events-in-entity-component-systems-22dc5107ed81) and
[What is an Entity Component System architecture for game development?](https://www.richardlord.net/blog/ecs/what-is-an-entity-framework)
are also very good articles.

The video [Game Engine Entity/Object Models](https://youtu.be/jjEsB611kxs) demonstrates both approaches
and their main differences. **Bobby Anguelov** explains beautifully why in practice there are serious issues with both
approaches.

To summarize the pros and cons of each:

### Game-object component model (EC)

In a Game-object component model nearly everything in the game world is modeled by a GameObject (also referred to as an Entity).
These game objects are then also ordered into spatial hierarchies, basically trees.
This is done so that dependent transformations can be easily managed. The easiest example to imagine is scene composition.
Lets say our character has two arms, two legs and a torso and a weapon (but no head!). Then you could have a Game-object
hierarchy like so:

```text
Character
  LeftArm
  RightArm
    Weapon
  LeftLeg
  RightLeg
```

Each object can have its own components, in this case some Mesh and some Transform (scaling, rotation and translation) data.
To make moving the character with all its parts easier we define each components transformations in their local space. 
The local space defines how to transform the object as if its parent was the center of the universe. 
The right hand rotates, the weapon also rotates because it is under the right hand.The character moves, every
part of it moves.

{{< figure src="spatial_hierarchy.png" >}}

Then to know where every game object is, we simply have to walk through the tree and multiply all the matrix transformations together and it is done.
Simple. On the other hand if you had to define and maintain every objects transformation in world space that would be a nightmare.
This gives the developers a simple mental model to work with, which is also very flexible and extensible. Basically everything in a scene
can be defined as a hierarchy of game-objects, put together from the required number of components.

Unfortunately this where all the positives end as well.

The idea is only simple as long as there is nothing more complicated then strictly hierarchical updates, where everything is only affected by the
changes to its immediate parent. In practice this is never the case.
Just imagine an enemy trying to find a target. What should it do? Walk the whole tree up and down and see if it can see some adversary? Perhaps ask
some higher level service to find it a target? Oh oh, but then the logic is no longer encapsulated within the component.
In either case it had to have access to a bigger scope than itself, i.e. it has escaped its encapsulation again. This introduces implicit dependencies between objects and components.
Or how about the physics/ai simulation? The logic cannot be encapsulated within a component, because they have to be global simulations.
Just imagine the AI is moving units around to attack. If each unit makes a decision individually then they will likely rush the closest target or
they engage into some complicated song and dance where they discuss who does what (which either introduces dependencies between the two or another
higher level object). If instead a global AI makes the decisions then it can internally decide what to do, but only if it knows about every unit it
controls. Is that unit still there though? Who triggers its processing? A unit? Either way you can easily see that no matter how you approach it
there are aspects which cannot be encapsulated within a component.

It also seems that with this pattern there is no solution to the problems. Encapsulation has to escape the objects/components if their
logic wants to do anything useful.
In practice a spaghetti, dependency/reference hell ensues and hell comes to Earth.

Moreover it is nearly impossible to reasonably parallelize any of this. There is no way to tell who isn't affected by what if the dependencies are implicit.
The moment the objects start accessing the world, walking the tree, anything can happen.

### Entity component system (ECS)

The Entity component system flips everything on its head and seems to seemingly say "What if we completely detached data from its behavior?".
A fascinating idea with data oriented design at its center. In this model this is what we will have: **World**, **Entity**, **Component**, **System**.
An **Entity** turns into nothing more than an ID. **Components** only hold raw data, no behavior at all. A component doesn't even know which entity it
belongs to. All entities are grouped into **Archetypes**, which are managed by the **World**.
In monkey terms this means that the World holds access to Archetypes which are simply tables for all entities that have the exact same set of components.
A sorta pseudo DB if you will.

{{< figure src="ecs.png" title="ECS components" >}}

The core idea is that all components (`ComponentA`, `ComponentB`) are held in continuous arrays tightly packed in memory. The Archetypes are nothing
more than groups of components. They don't really have a name, only tagged them for easier identification. Then an entity is just a given row across
components in an archetype.

Entities travel between archetypes when new components are added or removed from them. An entity will only be part of one archetype, with all its
components.

Then all behavior is defined by **Systems**. Systems are not concerned with what they are working on. Only that the appropriate components are present.
In this case lets say a System needs everything with `ComponentA` and `ComponentB`. It queries the World for these parameters and it will receive
entities with ids: 1, 3, 11, 9, ..., 6, 4, 2, ... Notice that it receives all entities that contained the required components. Some had more?
That is of no concern. Looks like a duck? It is a duck, it doesn't need to quack.

A marvelous idea! Divide the data into the smallest sensible units, put them into arrays, then let systems access these arrays and process them.
In terms of data locality and cache friendliness you could hardly wish for better.

Parallelizing system evaluation looks simple, just run the inner for loop distributed between threads and done.

At least this is what the idea is. It looks like a good idea and for a number of days I was convinced it is a good approach for a game engine.
What else could you want? This is where research pays off. Often people tend to hype up the benefits of a given approach while minimizing the
shortcomings.

In a purely ECS approach every property of an entity has to be encoded within the data. This may be just a zero size type, which by its presence conveys
some meaning but it has to be managed. When an entity either gets a component added or removed it has to move between archetypes. The whole row
in the table has to be moved into another table, which can take time and requires additional indirections for performant execution.

The model struggles with the handling of events. Again the event somehow has to turn into data, added to an entity, which shuffles it around.
For similar reasons state machines are problematic as well.

Then you also loose the spatial hierarchy that is the cornerstone benefit of the Game-object component model. Modeling parent, child relationships
becomes difficult in code and vastly more difficult for developers to reason about.

[Sander Mertens](https://ajmmertens.medium.com/) proposes solutions to these shortcomings (like relations, entities as components), but after being presented
with all the information it seems that perhaps they are a bit forced. Too many support structures and indirections have to be introduced to be able to handle all
of these common scenarios.

## Which way?

As mentioned before I have zero practical experience with either model. Making it very difficult to make a choice. What we know for sure is that
the **Game-object component model** works. It has been around for a long time and most games used it through Unreal, Unity or other engines.
The **Entity component system** is a newish approach. It has been around for a while now though. Without pretending to be a historian we can assume that it has
been used, the very minimum, at least five years. [Games have been made](https://github.com/SanderMertens/ecs-faq#who-is-using-ecs) using the pattern.
How accurate these reports are, did they exclusively use the pattern, I cannot say though.

As an inexperienced observer I tended to agree with the points made in [Game Engine Entity/Object Models](https://youtu.be/jjEsB611kxs). Most likely
we should not attempt to do either this or that, but strive to use the best of both words. The ideas outlined in the video seem reasonable to me.

Not this time though! While I have put all this at the beginning of the post, the research was the last thing I have done. Highlighting this as the
issues addressed in the previous sections were the ones that seemed to be beyond my capabilities.
No matter from which angle did I try to attack the design problem it always quickly fell apart. It never occurred to me that to even start the design
it was necessary to think so outside the box, that we can see a neighbouring one as well.

In the next chapter the goal will be to start and arrive at something described by **Bobby Anguelov**.

## What has been done

Funnily enough the only part that I could make progress on isn't really addressed by the previous sections.
How to detach a rendering API, how to feed the data to the GPU and how to think about rendering pipelines.

From the previous escapades it was understood that `fewer polygons => faster`, `fewer draw calls => even faster`.
So my goal was to try and find a pattern for writing the rendering pipelines that could allow me for supporting LOD (level of detail) with instancing.
Before that though, it was imperative to understand the nature of a rendering pipeline and what it means. Or how to think about it in such an abstract
way that it is helpful, but not overcomplicated.

### Rendering pipeline

This is not a formal definition of a rendering pipeline. Don't know if one exists, this is just how I started to think about it, after refactoring the
code 3-4 times and finally realizing what was before me.

There are 3 main components to a rendering pipeline:
- shader codes
- resources
- bindings

**Shader code** - is simply the program you may write to be used within the configurable stages of the GPU's rendering execution. In our case only "vertex" and
"fragment" shaders, but there can be more. These shaders define an interface for the resources they need through their bindings.

**Resources** - Most resources can simply be thought of as simple regions of memory on the GPU side. Any type that the GPU has built in support for like
floats, integers, vectors, matrices, arrays and structs can be combined together in any way. The GPU won't care. From it's perspective these will be nothing
more but a chunk of memory. As long as this chunk of memory follows the GPU's memory alignment requirements according to the defined types, the GPU will
happily consume the data.
There are some other resources though for which some stricter requirements are set. Textures and Samplers behave as completely black box resources. The programmer
does not know how they look like in GPU memory, cannot create them, only configure them (and in case of textures also upload the data, but you won't know
how it will actually look like in memory on the GPU side). These resources have to stand alone, with their individual bindings and cannot
be aggregated into arrays. Not sure why exactly this is, or if this is the case for all APIs, for WebGPU it seems to be the case.

**Binding** - There are two sides of a binding. One on the shader side and one the pipeline side. The shader side can be imagined as the socket and the pipeline side the
key. As long as they match everybody is happy.

{{< figure src="pipeline_pieces.png" title="Pipeline" >}}

From the CPU side a blob of data (can be anything, structs, arrays, can be at one place or coming from multiple) is taken, serialized according to the format of the bindings
it is made for, then uploaded to the GPU. At this point we got a handle to a GPU resource, in the example a `Buffer`.

Then a `Rendering pipeline` is nothing more than the collection of `shaders` and a collection of `Bind groups`. How one `Bind group` looks like is defined by the given group's
layout. How `Bind groups` are grouped together is defined by the `Pipeline's layout`. To then execute the `Rendering pipeline` we only need to create a `Bind group` that
is compatible with the required layout, slot it in, then request the pipeline's execution.

Went to the trouble of showing this to highlight some key observations. After a `Pipeline` is constructed, it can simply be stashed away. Nothing needs to change on it as long
as it is in use. It describes exactly what should happen with the data, where should it look for it, where should it put the output. It is complete.

The only thing that may change between rendering cycles are the `Buffers`, `Textures`, `Samplers`, simply the resources. In my case, only the content of the `Buffers` change between
each cycle. To update a `Buffer`, the appropriate data has to be serialized and uploaded again. In case the allocated `Buffer` has enough space, it does not even need to be
reallocated, just overwritten. In case more elements are needed, we can reallocate the `Buffer`, in which case the `Bind group` has to be updated as well.

Rebuilding a `Bind group` seems to be relatively cheap. Creating new `Buffers` and serializing/uploading to the GPU seems to be the most costly
part of the operation.

### Instances, meshes, globals

One question that remained was how should shaders look like? Should they receive data as `uniform buffers`, `storage buffers`, get mesh data from `vertex/index buffers`?
It really wasn't clear to me what was the sensible approach, so a number of combinations have been tried out (in previous chapters), but ultimately if we want to have **instancing**,
there seems to be only one pattern that made sense.

#### Put all data that should be the same within a given rendering cycle as a global uniform buffer

This then can be as big as necessary and contain all the data that may be required by **any**
of the pipelines

```wgsl
struct Globals {
    view_m: mat4x4f,
    view_projection_m: mat4x4f,
    view_world_position: vec3f,
    // This inverse must not contain the transition
    // from view matrix, only the rotation and scaling. The projection
    // should be left untouched.
    translation_free_inverse_view_projection: mat4x4f,
};
```

Any being the operative word here. Currently 5 shaders are in use (normal_debug, normal_debug_wireframe, cube_map, textured_draw, skybox), neither uses all 4 of these values. **normal_debug** and **normal_debug_wireframe** use `view_m` and `view_projection_m`,
**cube_map** and **textured_draw** only uses `view_projection_m`, and **skybox** only uses `translation_free_inverse_view_projection`. This means that instead of maintaining and updating a uniform buffer for (view_m, view_projection_m),
(view_projection_m) and one for (translation_free_inverse_view_projection) individually, only one needs to be updated and only once per rendering cycle. (Haha just noticed `view_world_position` isn't used any more, enjoy.)

Instead of 3 uploads to the GPU we get away with only one. While this means that the size of this combined uniform buffer will never be optimal for a given pipeline, the reduction of total memory transfer is likely worth the tradeoff.

> If you check the code, you won't see this pattern used yet, because the Skybox shader and the Camera are tricky. The camera is needed for the whole rendering cycle and the skybox is related to it, but otherwise
uses no meshes at all. Instead of trying to force it in, it was left separately. When we have a better design for Entities, this should be cleaned up.

#### Put all the mesh data with LOD levels for a given geometry into its own buffer

{{< figure src="lod.png" title="LOD levels" >}}

```wgsl
struct VertexData {
    position: vec4f,
    normal: vec3f,
    uv: vec2f,
}

@group(0)
@binding(2)
var<storage, read> vertex_data: array<VertexData>;
```

This works whether or not a given geometry has multiple levels of detail. Simply have the most detailed layout at the front of the buffer and the decreasing resolutions to the back. Likely there will be no need to generate levels of detail
on the fly or modify the base geometry. So we can probably assume that as long as a given mesh is needed, likely its corresponding levels of details will be needed too.
While currently we don't have this implemented, it does not seem to be a far fetched idea. The player/camera moves around the scene a lot, causing constant changes in the required LOD. The only problem with this assumption may come when there is simply
not enough GPU memory left. That optimization requirement though is far enough that it is not of concern as of now. 

The benefit is that this information has to be uploaded to the GPU only once per scene. Then it can be kept solely in GPU memory, freeing RAM for other tasks. It could even be reused between scenes if it
persists. A `storage` buffer has a maximum size of at least 128 MiB, but higher sizes can be requested based on the GPU. The current `VertexData` takes up 48 B, so we will have space for approximately 2.8 million vertices. Mayhaps not enough for
`AAA` high definition models and all their LODs, but likely more than enough for everything else.

#### Put every instance specific value into another storage buffer

```wgsl
struct Instance {
    model_m: mat4x4f,
    normal_m: mat3x3f,
}

@group(0)
@binding(1)
var<storage, read> instances: array<Instance>;
````

This allows us to group all instances together and update only the values that change between rendering cycles. With all three patterns put together,
the whole rendering pipeline collapses into:

```text
start rendering
update global uniform buffer
for each pipeline:
  for each group:
    update instance buffers
    set pipeline
    set bind group
    draw
end rendering
```

In other words only as many draw calls have to be made as pipeline and groups are in use/necessary.

#### Groups

How do you group draw requests for efficient instancing? We have to remember the GPU has to be told at the `draw` command how many
vertices does it need to draw and how many times to repeat the whole process (instances).

Differently, at the draw call, we must know, which mesh, with which LOD is being drawn, to know exactly how many vertices to be drawn.
This will be universally that same restriction for each (pipeline, mesh, LOD) combination. So our minimum grouping criteria is a match
on such triplets.

Then, depending on the pipeline and the used resources, the grouping criteria must be extended.
Remember this is necessary, because some resources, like Textures and Samplers cannot be put into arrays/`Buffers`. They always have to
stand as their own resource. Which means that if between two entities a different Texture is used, while everything else being the same,
they cannot be instanced together.

No special resources:
```text
Pipeline A doesn't require special resource to be bound.

# Instanced group 0
mesh 0, lod 0, Pipeline A, Instance{}
mesh 0, lod 0, Pipeline A, Instance{}
mesh 0, lod 0, Pipeline A, Instance{}
...

# Instanced group 1
mesh 1, lod 0, Pipeline A, Instance{}
mesh 1, lod 0, Pipeline A, Instance{}
mesh 1, lod 0, Pipeline A, Instance{}
...
```

With special resources:
```text
Pipeline B uses a Texture for rendering too, which cannot be serialized into a Buffer, necessitating another group.

# Instanced group 0
mesh 0, lod 0, Pipeline A, Texture 0, Instance{}
mesh 0, lod 0, Pipeline A, Texture 0, Instance{}
mesh 0, lod 0, Pipeline A, Texture 0, Instance{}
...

# Instanced group 1 
mesh 0, lod 0, Pipeline B, Texture 1, Instance{}
mesh 0, lod 0, Pipeline B, Texture 1, Instance{}
mesh 0, lod 0, Pipeline B, Texture 1, Instance{}
...

# Instanced group 2
mesh 2, lod 0, Pipeline B, Texture 0, Instance{}
mesh 2, lod 0, Pipeline B, Texture 0, Instance{}
mesh 2, lod 0, Pipeline B, Texture 0, Instance{}
...
```

All this means that apart from the default grouping requirements (mesh, lod, pipeline), the pipeline itself may enforce additional grouping criteria.

## Engine state

A number of refactors have been attempted in this iteration. A basic configuration system was added, which isn't used apart from reading the desired
resolution.

An attempt was made do handle nodes and the camera as some uniform entity, but this has failed given the complexity regarding entities.
Nevertheless the rest of the nodes were extracted from the pipelines and are now stored in an array. The idea was to simulate a scene tree in
the simplest way possible, where there are no hierarchies. This way we could test out what it would look like to feed every object that needs to
be rendered, every cycle to the rendering pipeline. To facilitate this pipelines now automatically group and instance all render commands.
Which enables us to dynamically change rendering related parameters for any entity with very little performance impact.
This was the only success. In hindsight we are not entirely sure how beneficial this feature really is, at least we have explored it.

The takeaway is that game engines are very complicated and even well established approaches aren't bulletproof. Regardless, we managed to get a bit
closer to the desired outcome.

(Increased the rendering resolution from 1024X768 to 1920x1080, this barely affects the FPS.)

{{< youtube 4hGfZbY_I80 >}}

Code available at: [v0.4](https://github.com/TheMrAI/engine/releases/tag/v0.4)

### FPS counter

In the previous chapter we had the following FPS metrics:

`develop` build
```text
Avg. FPS: 1957.49
1% low: 1850.92
0.1% low: 1785.03
```

`release` build
```text
Avg. FPS: 4728.06
1% low: 1564.94
0.1% low: 1539.11
```

Now we have:

`develop` build
```text
Avg. FPS: 905.07
1% low: 709.60
0.1% low: 603.94
```

`release` build
```text
Avg. FPS: 3027.90
1% low: 1190.20
0.1% low: 1003.72
```

As it can be seen we have lost a lot of FPS. This may be okay, given that we have now gained the minimum flexibility required in rendering
anything on demand. Furthermore the prior implementation had no chance at being parallelized. Now that may not be so far out of reach, in
which case we may get back all the lost processing power and perhaps more.

### Blooper

During refactoring obviously we messed up some buffer offsets. Here is one of the results.

{{< figure src="refact_blooper_1.png" >}}
