---
layout: page
title: Fume extractor
draft: true
---

## The problem

Not so long ago I have started 3D printing and even before that I have been dabbling in electronics, which
entails soldering. For a long time I have been aware that soldering releases dangerous fumes, but that
somehow didn't bother me too much, because I simply don't solder that often.
However, after starting 3D printing and doing a few 6+ hour prints it became evident that something gets
released into the air. You have this hard to describe smell that feels unnatural. I hesitate to describe it
as "plasticky", because it doesn't even remind me of that, it is more like air pollution. Similar to what you
would experience breathing in the air in a crowded city center, right next to some high traffic road.

The easiest solution to do would be to simply buy a fume extractor, but these start from 500-600 USD or more
depending on the brand and type. A little too steep for the hobbyist.

## Idea

How about we make a cheap one from garbage? Yeah, that sounds more like it!

According to [Marco Reps](https://www.youtube.com/c/MarcoReps) there really shouldn't be too much to it.

{{< youtube id="nJNqxeDSNpY?start=504" autoplay="false" title="All New Everything - Soldering Bench Upgrade for Productivity Ep1" >}}

Just a vacuum cleaner motor, some hoses and some filtration. The filtration material should be activated charcoal
(could be bought from your local pet store for fish tank filters), HEPA filter (from any electronics company
selling vacuum cleaner parts) and some hoses, some motor housing and all should be simple (My oh my was I wrong
about that).

{{< figure src="idea.jpg" title="Idea mockup drawing">}}

## Construction

The hose, its connecting elements should be relatively simple, then we can work on the motor housing, the impeller
designs to make generate negative pressures for the suction action, and then we just need to put it all together.

### Hose and connections

For the hose a simple common air duct piping was chosen. It is flexible, cheap and easily available in any
home improvement store.

{{< figure src="air-duct.jpg" title="Air duct">}}

This type of piping is perfect, because it comes in many sizes and one of the common ones has a diameter of 102 mm,
which is very close to the most common axial fan size of 125 mm. So in the future if we may need to add some such fans to
increase the CFM along the piping we could do it.
One potential problem with this piping is that it doesn't have a nice airtight fitting for it. That is however, something
we can mitigate with some 3D printing.

The idea is to use the helical shape of the piping and make such a fitting that can be screwed into it. The other end of this
piping should be compatible with a 125 mm axial fan mount. According to my measurements the internal diameter of this
standard piping is 90 mm while the external one is 110 mm (not quite sure what the 102 mm is supposed to measure in the
specification).

{{< figure src="air-duct-connection-version-1.jpg" title="Air-duct connection version 1" >}}

{{< figure src="air-duct-connection-version-1-fit-test.jpg" title="Air-duct connection version 1 fit test" >}}

Well, this didn't work out. While the general dimensions were correct the girth of the spiral was way too much, resulting
in an alarmingly tight fit. It fits, but it feels like it is going to burst at any moment.

{{< figure src="12cm_fan_to_airduct_v7.png" title="Air-duct connection version 2 model">}}

{{< figure src="air-duct-connection-version-2.jpg" title="Air-duct connection version 2" >}}

{{< figure src="air-duct-connection-version-2-fit-test.jpg" title="Air-duct connection version 2 fit test" >}}

After reducing the spirals' thickness it fits perfectly.

Fusion 360 files: [Air-duct connection](https://drive.google.com/file/d/1xlnxEKTH6QuJ732jFBLrokHPp-Z7FVVv/view?usp=sharing)

### Motor housing

This in itself was a hoot and a half! You would think, what can be hard about measuring out the dimensions of motor
components and then just simply making a housing for it that would hold it together? For one, it turns out measuring
sub-millimeter distances isn't easy, and when it counts, that is kind of a bummer. This saga can be inspected at
[motor housing journey]({{< ref "/tinkering/motor_housing" >}})

{{< figure src="motor_housing_version1" title="Motor housing version 1" >}}

### Impeller design

### Filtration

### Putting it all together
