---
layout: page
title: Voxon
summary: "A mixed principle game engine."
draft: true
---

## Why

Pff, why another game engine?! You would rightly say. It is too much work
to make it properly functional so that it ever has a hope of being
usable. Why bother?

Simply put, because I find it interesting and it is extremely complicated.
The amount of mathematical/programing wizardry that goes into making an
efficient game engine is simply staggering.
It is quite astonishing to learn how the GPU works, why some algorithms
are efficient why others aren't at all, even if in theory they could be good.
The problem space is simply complex enough that you learn that being a half
decent programmer isn't necessarily about patterns, clean code or any other
marketing term that goes around, but the hard choices you have to make for
achieving the desired results. Is it getting too complicated? Then you need
the discipline to still make it understandable.

Compilers, transpilers, math, multithreading, GPU programming, design, research,
everything you can think of is in there. There is just nothing like it!
And the best thing? If you make something work you can render it, you can show it
off, you can share it with others!

Secondly, I think everyone needs to be crazy about something or we will go mad.
Some people do that with their collections, garden, family. Mine will be game
engines.

Furthermore I don't think the industry is going in the right way. Unoptimized blurry
slop seems to be the name of the game, so that newer and newer graphics cards may be
sold while also decreasing clarity. Consumption must continue!
The biggest offender seems to be [Unreal Engine](https://www.unrealengine.com), the 
insistence on raytracing, frame generation and upscaling. Sure, let's use some tricks,
but my lord we should at least try to make it look good.

## The goal

Is to make a game engine while understanding most prior established methodologies.
The tips and tricks that the industry has developed over the decades.
This engine should run on all major platforms: Windows, Linux and Mac.
Should be as efficient as reasonably possible, produce readable documented code
which can later on be used as a resource for others.

Most importantly it should explore how major ideas may be used in
conjunction efficiently.
Mesh based topologies or signed distance fields, voxels?
Raytracing or rasterization maybe something else?
There are so many great ideas out there all with their strengths and weaknesses.
Maybe we can merge them together somehow and make something even better?

The code can be found here: [Voxon](https://github.com/TheMrAI/engine)

It is primarily written in Rust. The reason why Rust and not C++ was chosen is, because
Rust is a more user friendlier C++. While it is still an extremely complicated language,
at least in Rust one can afford to be a little less paranoid than in C++.

The road:
