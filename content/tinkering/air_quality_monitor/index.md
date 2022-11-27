---
layout: page
title: Air quality monitor
date: "2022-11-26"
summary: "A simple air quality monitor, measuring temperature, humidity, air pressure, CO2 concentration
and displaying it on a simple LCD screen."
---

## Idea/goals

Sometimes I am cold, when the air doesn't feel cold. Am I getting sick perhaps? Sometimes, my
head hurts and I get sleepy. Is it because the air in my room got stale, and it would be nice
to refresh it or is it because a storm is coming and the air pressure either dropped or rose
suddenly?
There is simply no way to know without measuring it. So, why don't we make something simple that
can do just that and display to us what is the actual situation. Then, there is no need to wonder
and if needed I could react accordingly.

It would be nice to have an easy-to-read display, where the temperature, humidity, pressure and
CO2 concentration could be displayed. The whole thing should be a self-contained little unit that
could be moved around, without any power cords. There are already enough gadgets with power cords,
there is no need for any more.
Finally, because the whole thing would be in an enclosed little self-contained box, it would be nice
if we forced the air trough it, so we would not just rely on passive airflow for updating the measurements.

It would be a bonus if it could last as long as possible without the need for a recharge. That is annoying
and if we think about it a little there is really no need to measure constantly. We need measurements exactly
when we care about them.

(Maybe if it could act as a fidget toy as well, that could be a bonus too! Nice clicky switches for everything!)

## Design

### Parts

An [Arduino Nano](https://store.arduino.cc/products/arduino-nano) as the brain of the operations should suffice.
It consumes only 19 mAs which is very good. For the LCD lets have something cheap like this
[MC21609A12W-VNMLB](https://uk.farnell.com/midas/mc21609a12w-vnmlb/lcd-alpha-num-16-x-2-blue/dp/2483340?ost=mc21609a12w-vnmlb).
The pressure, humidity and temperature are all taken care of by this [BME280](https://www.bosch-sensortec.com/products/environmental-sensors/humidity-sensors-bme280/)
which seems to be readily available in small easy to use modules like [this one](https://uk.farnell.com/dfrobot/sen0236/i2c-environmental-sensor-arduino/dp/3517927?st=bme280).
The CO2 measurement could be done by something like this [SEN0159](https://uk.farnell.com/dfrobot/sen0159/analogue-co2-gas-sensor-arduino/dp/3517879?st=co2%20sensor) module,
and finally we need a tiny fan like the [EB40100S2-0000-999](https://uk.farnell.com/sunon/eb40100s2-0000-999/fan-40x40x10mm-5vdc-7cfm-27dba/dp/1924831?st=sunon%2040%20mm).

Of course, we need a battery too, I think a regular 18600 would suffice,
[a battery holder](https://uk.farnell.com/keystone/1043p/batt-holder-18650-x-1-th/dp/4049873?st=18650%20holder) for it and a
discharge protection circuit such as the
[AP9101CAK-ABTRG1](https://uk.farnell.com/diodes-inc/ap9101cak-abtrg1/battery-protector-li-ion-li-pol/dp/3483006?ost=3483006).

We could integrate a battery charging circuit, but as a start the goal is to have something simple, that I could build. To be honest at this
stage I have no real experience with battery management, I2C, LCDs, so if we can keep one complexity down that is a win.

> Please note, that the list of parts is not complete and are only provided as an example. The actual parts used should be decided mostly on what
is available and affordable.

## Mockup

TODO mockup

## Implementation

First we have to understand how our parts work and what are their input voltage requirements and current consumption rates.

### LCD

The exact LCD module I could get, has a [HITACHI HD44780U](https://www.sparkfun.com/datasheets/LCD/HD44780.pdf) driver, and it looks/operates
exactly the one in Ben Eater's excellent video explaining its usage here:

{{< youtube FY3zTUaykVo >}}

From the datasheet we can observe that this module has a low operating voltage between `2.7 - 5.5 V`. Upon some testing we can observe that
although the module itself will start operating at `2.7 V`, but the screen and backlight will only become really clear (strong color, with no flickering)
when the voltage reaches **5 V**. That is what we will target then.

The current consumption of the full LCD module is about **~13 mA**, according to my measurements. This means we can directly supply it from
the Nanos 5V PIN which can supply `40 mA` on all of its pins.

### BME280 sensor

On the sensors datasheet [BME280](https://www.bosch-sensortec.com/products/environmental-sensors/humidity-sensors-bme280/), we can see
`Supply voltage VDD: 1.7 - 3.6 V` with an average current consumption of **3.6 μA**.
Excellent! Similarly to the LCD we can directly supply power to the BME280 module through the `Nanos 3.3 V` pin.

### This is not CO2 sensor we seek

This is the point when reading the datasheets yourself instead of just trusting online junk really pays off. In my case I didn't buy the
exact sensor I have linked above but something a bit cheaper promising the same functionality, using an
[MQ-7](https://www.sparkfun.com/datasheets/Sensors/Biometric/MQ-7.pdf) sensor (Yes, this is a CO sensor not CO2, but depending on the
listing, once again, you can be fooled). What can we see in the data sheet?
During its operation it will have a heating phase where it needs `5 V` and consumes `350 mA` of current, and a measurement phase where it
only requires `1.4 V` for which no current was specified (we can assume something much less than 350 for sure).

This means that right of the bat, about half of the online sources are garbage for using this module.
[MQ-7 Carbon Monoxide Sensor Circuit Built with an Arduino](https://www.learningaboutelectronics.com/Articles/MQ-7-carbon-monoxide-sensor-circuit-with-arduino.php)
would ask you to connect it directly to the Arduino Unos `5 V` pin, which can only supply [20 mA](https://store-usa.arduino.cc/products/arduino-uno-rev3).
Killing your board very quickly. Just to show how prevalent this issue this is see two more examples: [one](https://create.arduino.cc/projecthub/ingo-lohs/carbon-monoxide-gas-sensor-mq-7-f7f327), [two](https://create.arduino.cc/projecthub/ingo-lohs/carbon-monoxide-gas-sensor-mq-7-f7f327).

The exact module I got was functionally equivalent to this one
[MQ-7](https://www.amazon.com/Rakstore-Carbon-Monoxide-Sensor-Detection/dp/B0B1CTRX4T/ref=sr_1_4?crid=15FZ5AP0CBYIP&keywords=mq-7+module&qid=1669546743&sprefix=mq-7+module%2Caps%2C641&sr=8-4). What can we remember from the data sheet? The sensor works with a heating phase, then during a measurement phase we
get our data. If you look at this board it has no active components apart from the
[LM393](https://www.ti.com/lit/ds/symlink/lm393-n.pdf?ts=1669468320254&ref_url=https%253A%252F%252Fwww.google.com%252F) voltage comparator. There is no
circuit on the board that could possibly reproduce the heating/measuring cycles required by the MQ-7 sensor. So the module itself quite simply
cannot be operational. If you would like to learn more about this, there was one online guide that investigated the situation, and it is worth a read
[here](https://www.instructables.com/Arduino-CO-Monitor-Using-MQ-7-Sensor/).

Regardless, the MQ-7, according to the manufacturer, requires a 48h calibration cycle, where you have to know the precise concentration of CO at all times.
For obvious reasons, this is not really feasible for a DIY project. Whatever results we may get, would be inherently unreliable and then there is simply
no point in measurements.

So first I have duped myself by buying a CO module instead of a CO2 one. Secondly, I trusted the manufacturer of the module without checking the datasheet first.
The module I have linked above for the [CO2 sensing](https://www.dfrobot.com/product-1023.html) is likely garbage too,
because I could find from them a [CO module](https://www.dfrobot.com/product-686.html) using the MQ-7 sensor which has the exact same issues described above.

### CO2 sensor

With the lessons learned from the previous failure I started looking for a substitute sensor and have found a promising one called
[XENSIV PASCO2](https://www.farnell.com/datasheets/3680038.pdf). Looks pretty good on paper with an operational range between 0 - 32000 ppm,
10 year lifetime and a 30 mW power consumption. The package size is 14 x 13.8 mm, and it has 10 pins in total. Based on my guesstimates (and the datasheet)
the smallest pin for this module has an area of about 2.7 mm^2. That is just at the limit of my hand manufacturing capabilities.

### Prototype