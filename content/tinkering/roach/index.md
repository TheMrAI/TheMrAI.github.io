---
layout: page
title: Roach
date: "2022-10-03"
summary: "Ever wanted to have a little robot that is so clumsy it is almost fun to watch? Here is my first
attempt at doing just that."
---

## The first idea

For a while now I wanted to have a tiny little robot that would just roam around the room, without much
purpose and try to mimic some behavior, that would make it somewhat amusing to look at.

Now, since I know very little about electronics, I wanted to set the initial goal very low. So you know it
can actually be achieved. I could imagine an awesome little spider-bot, but couldn't really build it.

So simple it is, what can be simply made? Let's make a tiny tank, as that is the simplest control method.
Attach to it an ultrasonic distance sensor, so it doesn't bump into too many things, and give it a
photodiode, so that it can detect light intensity and hurry up if it is brighter and chill out when it is
dark. Like a cockroach. Oh, and no batteries! No idea how to manage them, which type to buy, how much...

As a controller I will be using an [Arduino Uno Rev 3.](https://docs.arduino.cc/hardware/uno-rev3), with its
[motor shield attachment](https://docs.arduino.cc/hardware/motor-shield-rev3).

{{< figure src="roach_v1.png" title="Roach mockup" >}}

### Designing the carriage

After searching for a few Challenger 2 reference photos online, measuring, and scaling the dimensions to my
little Roach I dove right into 3D modelling. Did you notice the problem yet? I didn't either and happily
modelled away a few hours until I have realized. My, my I can't possibly manufacture a tank thread this small.
The rollers would be about 16 mm in diameter, 9.8 mm wide, their internal shafts would need to be smaller
than 1.5 mm. That would be almost doable, but the thread links could not possibly assemble, as they would
be even smaller, and they would need to be connected with tiny pins like a bicycle chain.

Okay, so tank thread out the window, what else to have simple tank like control? Get a small swivel wheel at
the front and two motorized wheels at the back. Then it would essentially be able to rotate as a tank in
place, without tank thread. Yay! It got a bit simpler too!

{{< figure src="carriage_v7.png" title="Carriage version 7" >}}

I didn't model the back wheels or the ultrasonic sensor mount, or the holes where the Arduino boards will be
mounted into. Too lazy and you can always drill plastic.

### Assembled

{{< figure src="roach_front.jpg" title="Roach front" >}}

{{< figure src="roach_front_top_right.jpg" title="Roach top right" >}}

### Test run and evaluation

{{< youtube MAGXDF7ToxY >}}

It goes around aimlessly, as designed. When faced with an object about 5 cm of it, it will reverse for
a moment, turn about 90 degrees in a random direction, then just go into the void.

It is sufficiently fun to watch it bumble around, so I believe we can call this a proof of concept and go to
the second version.
