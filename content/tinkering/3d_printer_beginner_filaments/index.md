---
layout: page
title: 3D printer beginner filaments
date: "2022-10-30"
summary: "A beginners experiences with the two basic filaments for FFD printers. On PLA, PETG and moisture.
If you are a first time 3D printer buyer and are faced with the decision on which filament to buy and why, this is
the post for you."
---

## Which filament to buy

As a beginner it is hard to decide which filament to buy or how to start out.
There is an incredible amount of material online describing the benefits and downsides of each filament type.
For example this [table](https://help.prusa3d.com/materials) on Prusa's page or the huge range of videos
made by [CNC Kitchen](https://www.youtube.com/playlist?list=PLEOQTmIWJ_rncRcWmjQIvMKFAeM071CXM) just on filament
strength.

Without loosing yourself in the depths of data, before you can make a choice, let me simplify the problem for you.
Buy **PLA** first and when you feel comfortable with that, you can try out **PETG** and go from there based on your needs.
Don't even bother with other filament types. Chances are, as a beginner you don't know how to run your printer. How to fix it
or how to troubleshoot it. More importantly you will be much happier if you can focus on printing first without having to
investigate what is wrong.

### [PLA](https://help.prusa3d.com/article/pla_2062)

#### Pros

The bog-standard printing material and with a good reason! It is one of the cheapest ones out there, even the cheapest printers
are able to print it. It is food safe (don't eat from the results) and has very good printing characteristics.
What does all this mean? It just works!

While printing the room won't smell like cancer, the result will almost certainly be very clean and very precise.
PLA doesn't string and is rather tolerant to humidity levels. In other words you don't have to worry about keeping your filament
dry. A normal room is at around 50-60% humidity and this won't affect PLA much at all.

Supports are easy to remove, it doesn't warp or shrink and doesn't detach from the heatbed.

On its mechanical characteristics one unfamiliar can get a bit confused. The materials impact resistance is one of the lowest ones out
there, but its tensile strength is high. Now what does this mean? A low impact resistance means it can't really consume energy by bending,
it will rather break. A high tensile strength means it handles static loads well. In other words you can make shelves, holders, models
out of it, but not gears for a functional gearbox.
Now in reality, most people don't buy a 3D printer to make impact resistant objects, but to have fun, print models to paint and/or use.
Develop a design, print a model that can be interacted with to validate it. For those applications PLA is perfect.

#### Cons

There are only two real downsides to PLA. Its heat deflection temperature is only at about 60 ℃, and it does not tolerate UV well.
60 ℃ seems like a low temperature, but most of us don't live in the Sahara, where the highest recorded temperature was only 58 ℃
(well only in the context of melting plastic, it would be rather toasty for a regular human).
So that may not be that big of a problem if we don't plan to print parts that are intended to be used in hot environments.

It doesn't tolerate UV, but in reality not many things do. A wooden patio will be destroyed if not maintained regularly within a few years
of exposure from the Sun. In day to day how much this would affect PLA is hard to say.
The only experiment I have conducted on this was to put out small PLA models to the balcony in the hottest summer days. A few into sunlight on
a black metal railing and another few into the shade. After a few days no observable change occurred to either batch. No warping, no discoloration,
nothing.

So unless, you would like to use your prints as something that would be regularly get exposed to UV and high heat, it should be fine.

### [PETG](https://help.prusa3d.com/article/petg_2059)

#### Pros

The bog-standard printing material number two?! Well yes. For about the same price of PLA it has similar characteristics with some improvements.
A little harder to print, requiring a bit higher temperatures, still well in the limits of cheap 3D printers. It is food safe, but still you shouldn't
eat from it.

During printing, it produces even less smell in my experience than PLA. It doesn't even smell to be honest, it is more like the air feels dusty (a sign of
airborne microparticles, not good to breathe for long so always ventilate).

Its mechanical characteristics are simply overall better than that of PLA. Higher heat deflection temperature at about 80 ℃, higher impact resistance and about
the same tensile strength. It isn't UV resistant but tolerates it much better than PLA.

In general, it is good enough that this is what many 3D printer manufacturers like Prusa use to print the parts for their printers. With a fine-tuned
configuration, prints will be clean and sturdy. Almost good enough for demanding mechanical loads like the functioning gearbox, but not quite. But even for
those scenarios it is good enough for small experiments.

What is the catch?

#### Cons

While the printed result tolerates moisture very well, this isn't the case before printing. While PLA can handle a normal room humidity PETG won't. Unless
it is dry as possible, the printed result will have a number of issues.

It will be stringy (small strings will appear between parts where the printer head had
to reposition, but shouldn't print).

It is harder to remove from the heatbed. Contrary to PLA it is very difficult to remove a print while the heatbed is still hot. Similarly, removing support
material is very difficult. The resulting supports bond much more to the print than for PLA. Sometimes enough that breaking them off can rip a few layers
off from the printed object itself.

### The effects of humidity

First, let's take a close look at how does it affect the results to print the same components with a dry or a moist PETG filament. In the following examples
the exact same filament type, exact same spool was used to print the same components, using the same G-codes.

#### Dry vs moist PETG results

{{< figure src="dry_vs_moist_1.jpg" title="Moist (left) vs dry (right) example 1">}}

{{< figure src="dry_vs_moist_2.jpg" title="Dry (left) vs moist (right) example 2">}}

The moist filament produced some very fuzzy results, completely merging the layer lines together and giving us this dirty looking texture. While the dry filament
produced clean layers, a clean/uniform color with a nice and shiny finish, while also retaining some transparency that the filament originally had.

The support materials in both cases adhered a little too much, ripping off 1-2 layers from the parts, but the effect was less significant in the case of the dry
filament. No pictures from these as I simply can't take clear enough images of them with my current camera.

#### Humidity tolerance between PLA and PETG

In the following pictures the first row used the same PETG filament. From left to right, the first is my test coin with a fresh out of the box PETG filament
(my first PETG print), the second using the same filament after a month left in my room and the third after drying out the filament.
On the bottom row, everything is printed with PLA. From left to right, the first one is my very first coin print, while the second one uses a PLA filament similarly
left to brace for room humidity for months on end. In this case there was no third print as I didn't try to dry the filament.
Additionally, the first coins from the top and bottom used a 0.4 mm nozzle (smaller extrusion lines), while the rest used a 0.8 mm nozzle.

{{< figure src="overall_comparison_top.jpg" title="Dry (left) vs moist (right) example 2">}}

{{< figure src="PLA_humidity.jpg" title="PLA humidity exposure effects">}}

{{< figure src="PETG_humidity.jpg" title="PETG humidity exposure effects">}}

There are 3 things we can observe regarding filament moistness. PLA isn't impacted, or the result is small enough that it isn't noticeable. The only thing
that is clear is that the 0.8 mm nozzle width seems to be a bit too rough for printing this small coin and some details are simply lost.
Second, PETG fresh out of the box was already vet enough that it has experienced some negative effects.
Thirdly, the dry PETG has the same quality (the picture isn't clear enough, my apologies) as the PLA.

### Conclusion

If you want to print small intricate models or bigger complex ones with lots of support material, but you don't need the mechanical capabilities of PETG then
use PLA. The filament is easier to keep and the results are easier to clean up.

If you need to print more functional parts, and you can do this with minimal to no support, then use PETG, but keep it as dry as possible.
