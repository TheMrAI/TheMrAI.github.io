---
layout: page
title: Cleanup
date: "2026-06-09"
summary: "Plans for the future and some refactoring"
draft:  true
---

Prior to this project I had close to zero experience (ehh call it an even zero) with game engine programming.
As a learning exercise a lot of mess was made. That is fine. Bumbling through the dark without knowing how to do
something inevitably leads to a mess. We learn from this mess and fix it in due time.

And this is where I got slapped in the face very hard. Had no misconceptions about game engines being simple in the slightest,
but nothing prepared me so far to the amount of sheer complexity and considerations that need to be made from the start.

As an exercise spent about 2 weeks just refactoring the code to make it simpler in the hopes of observing some patterns that
can be later used to build on. In some parts this was a success, in others an abject failure. Let me show you why.

## ECM or ECS

There seems to be two major approaches in taming the complexity spirit demon residing in every
game engine. Furthermore it seems it doesn't matter which one you choose, suffering is guaranteed, just in
different ways.

First of we have ECM - Entity component model, Entity system (ES), **Game-object component model**. Many names for the
same thing, confusing all non initiates. This is basically the object oriented approach.
[Game Programming Patterns](https://gameprogrammingpatterns.com/) by **Robert Nystrom** is a very good read in this regard.
Presents the patterns in a fun, understandable manner.
If you are as unfamiliar with the topics as I was, for this discussion it is recommended to read:
- [Game Loop](https://gameprogrammingpatterns.com/game-loop.html)
- [Component](https://gameprogrammingpatterns.com/component.html)
- [Event Queue](https://gameprogrammingpatterns.com/event-queue.html)
- [Data Locality](https://gameprogrammingpatterns.com/data-locality.html)
- [Spatial Partition](https://gameprogrammingpatterns.com/spatial-partition.html)
It isn't necessary to fully understand them, but at least observe what type of problems have to be solved.

Second is ECS, which has only one name that I could stumble upon, **Entity component system**.
This is the data oriented approach. A lot of good information can be found here: 
[Entity Component System FAQ](https://github.com/SanderMertens/ecs-faq#how-is-ecs-different-from-entity-component-frameworks)
To me the most helpful ones were:
- [Building an ECS #1](https://ajmmertens.medium.com/building-an-ecs-1-where-are-my-entities-and-components-63d07c7da742)
- [Building an ECS #2](https://ajmmertens.medium.com/building-an-ecs-2-archetypes-and-vectorization-fe21690805f9)
- [Building an ECS #3](https://ajmmertens.medium.com/building-an-ecs-storage-in-pictures-642b8bfd6e04)
- [Why Vanilla ECS Is Not Enough](https://ajmmertens.medium.com/why-vanilla-ecs-is-not-enough-d7ed4e3bebe5)
- [Building Games in ECS with Entity Relationships](https://ajmmertens.medium.com/building-games-in-ecs-with-entity-relationships-657275ba2c6c)
- [Making the most of ECS identifiers](https://ajmmertens.medium.com/doing-a-lot-with-a-little-ecs-identifiers-25a72bd2647)
- [Why Storing State Machines in ECS is a bad idea](https://ajmmertens.medium.com/why-storing-state-machines-in-ecs-is-a-bad-idea-742de7a18e59)
All of them written by [Sander Mertens](https://ajmmertens.medium.com/).
[Events in Entity Component Systems](https://medium.com/@casual.evil52/events-in-entity-component-systems-22dc5107ed81) and
[What is an Entity Component System architecture for game development?](https://www.richardlord.net/blog/ecs/what-is-an-entity-framework)
are also very good articles.

The video [Game Engine Entity/Object Models](https://youtu.be/jjEsB611kxs) demonstrates both approaches
and their main differences. **Bobby Anguelov** explains beautifully why in practice there are serious issues with both
approaches.

To summarize the pros and cons of each

### Game-object component model (EC)

In a Game-object component model nearly everything in the game world is modeled by a GameObject (also referred to as an Entity).
These game objects are then also ordered into spatial hierarchies, basically a tree.

[TODO example tree]

This is done so that dependent transformations can be easily managed. The easiest example to imagine is scene composition.
Lets say our character has two arms, two legs and a torso and a weapon (but no head!). To make moving the character with all its parts easier
we define each components transformations in their local space. The local space defines how to transform the object as if its parent was the
center of the universe. The right hand rotates, the weapon rotates also rotates because it is under the right hand. The character moves, every
part of it moves.

[TODO example]

Then to know where every game object is, we simply have to walk through the tree and multiply all the matrix transformations together and it is done.
Simple. On the other hand if you had to define and maintain every objects transformation in world space that would be a nightmare.
This gives the developers a simple mental model to work with. Unfortunately this where all the positives end as well.

The idea is only simple as long as there is nothing more complicated then strictly hierarchical updates, where everything is only affected by the
changes to its immediate parent. In practice this is never the case.
Just imagine an enemy trying to find a target. What should it do? Walk the whole tree up and down and see if it can see some adversary? Perhaps ask
some higher level service to find it a target? Oh oh, but then the logic is no longer encapsulated in it. In either case it had to have access to
a bigger scope than itself, i.e. it has escaped its encapsulation again. This introduces implicit dependencies between objects and components.
Or how about the physics/ai simulation? The logic cannot be encapsulated within a component, because they have to be global simulations.
Just imagine the AI is moving units around to attack. If each unit makes a decision individually then they will likely rush the closest target or
they engage into a some complicated song and dance where they discuss who does what (which either introduces dependencies between the two or another
higher level object). If instead a global AI makes the decisions then it can internally decide what to do, but only if it knows about every unit it
controls. Is that unit still there though? Who triggers its processing? A unit? Either way you can easily see that no matter how you approach it
there are aspects which cannot be encapsulated within a component.

It also seems that with this approach there isn't much do to about the problem either. Encapsulation has to escape the objects/components if their
logic wants to do anything useful.
So in practice this is what will happen. A spaghetti, dependency/reference hell ensues and hell comes to Earth.

Moreover it is nearly impossible to reasonably parallelize any of this. There is no way to tell who isn't affected by what if the dependencies are implicit.
The moment the objects start accessing the world, walking the tree, anything can happen.

### Entity component system (ECS)

The Entity component system flips everything on its head and seems to seemingly say "What if we completely detached data from its behavior?".
A fascinating idea with data oriented design at its center. In this model this is what we will have: **World**, **Entity**, **Component**, **System**.
An **Entity** turns into nothing more than an ID. **Component**s only hold raw data, no behavior at all. A component doesn't even know which entity it
belongs to. All entities are grouped into **Archetype**s, which are managed by the **World**.
In monkey terms this means that the World holds access to Archetypes which simply are table for all entities that have the exact same set of components.

[TODO example]

Then all behavior is defined by **System**s. Systems are not concerned with what they are working on. Only that the appropriate components are present.
For example a MeshRenderingSystem will request the TransformComponent and the MeshComponent. Every entity that contains these components will be
rendered by the MeshRenderingSystem. In other words systems work on and solely identify their entities based on the set of components they poses.

[TODO example]

A marvelous idea! Divide the data into the smallest sensible units, put them into arrays, then let systems access these arrays and process them.
In terms of data locality and cache friendliness you could hardly wish for better. For raw processing continuous memory is the best.

Parallelizing system evaluation looks simple, just run the inner for loop distributed between threads and done.

At least this is what the idea is. It looks like a good idea and for a number of days I was convinced it is a good approach for a game engine.
What else could you want? This is where research pays of. Often people tend to hype up the benefits of a given approach while minimizing the
shortcomings.

In a purely ECS every property of an entity has to be encoded within the data. This may be just a zero size type, which by its presence conveys
some meaning but it has to be there. When an entity either gets a component added or removed it has to move between archetypes. The whole row
in the table has to be moved into another table, which can take time and requires additional indirections for performant execution.
The model struggles with the handling of events. Again the event somehow has to turn into data, added to an entity, which shuffles it around.
For similar reasons state machines are problematic as well.
Then you also loose the spatial hierarchy that is the cornerstone benefit of the Game-object component model. Modeling parent, child relationships
becomes difficult in code and vastly more difficult for developers to reason about.
[Sander Mertens](https://ajmmertens.medium.com/) proposes solutions to these shortcomings, but after being presented with all the information we
can see that perhaps they are a bit forced. Too many support structures and indirections have to be introduced to be able to handle all of these
scenarios.

## Which way?

As mentioned before I have zero practical experience with either model. Making it very difficult to make a choice. What we know for sure is that
the **Game-object component model** works. It has been around for a long time and most games used it through Unreal, Unity or other engines.
The **Entity component system** is a newish approach. It has been around while now. Without pretending to be a historian we can assume that it has
been used, the very minimum, at least five years. [Games have been made](https://github.com/SanderMertens/ecs-faq#who-is-using-ecs) using the pattern.
How accurate these reports are or they mean they exclusively used the pattern, I cannot say.

As an inexperienced observer I tended to agree with the points made in [Game Engine Entity/Object Models](https://youtu.be/jjEsB611kxs). Most likely
we should not attempt to do either this or that, but strive to use the best of both words. The ideas outlined in the video seem reasonable to me.

Not this time though! While I have put all this at the beginning of the post, the research was the last thing I have done. Highlighting this as the
issues addressed in the previous sections were the ones that seemed to be beyond my capabilities.
In the next chapter the goal will be to start and arrive at something described by **Bobby Anguelov**.

## What has been done

Funnily enough the only part that I could make progress on isn't really addressed by the previous sections.
How to detach a rendering API, how to feed the data to the GPU and why.
