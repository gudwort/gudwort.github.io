---
layout: post
title: Using BLTouch with Klipper and the Ender3
category: 3dprinting
description: How to make your BLTouch Sensor work with the Klipper firmware and the Crealty Ender3.
tags: 3dprinting klipper
---

I have been 3D printing for about a year now, its had its ups and downs, good times and bad.  Some times its the greatest thing, and sometimes I'm glad I don't keep a baseball bat near my 3D Printer, because I may not still be the owner of one if I did.  I know all printers are different and everyone has their reasons for why or what they do and use for the build surface.  I have the [Ender3](https://www.amazon.com/Comgrow-Creality-Ender-Aluminum-220x220x250mm/dp/B07BR3F9N6/ref=sr_1_1_sspa?keywords=ender+3&qid=1559158769&s=gateway&sr=8-1-spons&psc=1), its a great entry level printer for the price and I have spit out some amazing prints.  But one the most notorious issues with the Ender3 is the warped bed.  It seems like everyone has reported this issue.  And I think one thing all 3D Printer Hobbyist could agree on, is the first layer is one of the most important parts of the print.  And when you printing around .2mm layer hight, if the build surface isn't perfect you're going to have problems, especially with larger prints.

There are a few things you could do about this, such as, just deal with it, replace the bed with something higher grade, or add a new surface, such as a piece of glass.  I chose to use a sensor on the hot end.  I went with the [BLTouch](https://www.amazon.com/BLTouch-Leveling-Printer-Official-Authorization/dp/B07GVCX74T/ref=sr_1_4?keywords=bltouch&qid=1559158792&s=gateway&sr=8-4) sensor.  It's pretty much just a probe, that will make a _mesh_ of your build surface and allow for your Z axis to compensate for the imperfect surface with very small up and down Z axis movements.

Now, I know there is a lot of YouTube videos out there on how to setup and use a BLTouch sensor, but, everything I found when I was setting mine up, was with using the Marlin firmware. Either Vanilla Marlin or the TH3D flavor.  Well, to complicate things even more, I am using [Klipper](https://github.com/KevinOConnor/klipper).  I'm not going to dive into much about Klipper in this blog post, as its not really the focus.  Either you are already using Klipper and you know what it is, so keep reading, or go research Klipper and see if its for you.  Because these settings will **NOT** work with any flavor of the Marlin firmware.

## Values You'll Need
There are values you need when setting up your BLTouch, namely the X and Y offsets from the nozzle to the BLTouch, you can either measure these or, if you are using third party mods on your hot end, a lot of the designers will specify these values.  I am using the [Bullseye Cooling Fan Duct](https://www.thingiverse.com/thing:2759439) and the designer has provided the offset values on the Thingiverse page.  Using the Bullseye Fan Duct and the provided mount, the offsets are `X -42, Y -5`.

Next, you will need to know your bed size, the bed that ships with the Ender3 is `220x220mm`.  You need to know this for creating a bed mesh, which is what the BLTouch will build when probing.

## Get Configuring
Here are the settings you need to add or update on in your Klipper `printer.cfg`:
1. You need to change the `endstop_pin` value to use a virtual endstop, this is in the `z_stepper` section.
  * `endstop_pin: probe:z_virtual_endstop`
2. You need to allow the Z Axis to go below the '0' mark for negative warp, this can be done by editing the `position_min` value, also in the `z_stepper` section.
  * `position_min: -3`
3. You need to define the `[bltouch]` section and values in the configuration file (a section is defined with square brackets, this section does not exist in the Ender3 printer.cfg by default, so you will need manually create it).  You can see that the X and Y offsets are the values I got from the Thingiverse page.
```
[bltouch]
sensor_pin: ^PC4
control_pin: PA4
x_offset: -47.0
y_offset: -5.0
z_offset: 1.65
speed: 5.0
samples: 2
sample_retract_dist: 8.0
```
  * The `samples` value is how many times it checks each location in the bed mesh, more samples will take longer.
4. Next, the `[bed_mesh]` section needs to be manually defined.
```
[bed_mesh]
speed: 80
horizontal_move_z: 5
min_point: 50,30
max_point: 230,230
probe_count: 5,5
```
  * The `probe_count` is how many samples its going to take, `5,5` is a `5x5` grid, a bigger grid will take longer, but may be more accurate.
  * The `min_point` and `max_point` is how big your bed is, with the `220x220mm` build plate, `230x230mm` is about as large as you can go.
5. Now, add the `[homing_override]` section.  You can do what ever you want here, I am just setting the Z axis to home in the middle of the bed.
```
[homing_override]
set_position_z:0
gcode:
    G1 Z10 F600
    G28 X Y
    G1 X166 Y120 F6000
    G28 Z
```
  * PRO TIP: gcode sections in the printer.cfg need to be 4 spaced or you will get errors.

6. There is one last thing to do, and that is configure your start and end gcode sequences to tell klipper to generate the bed mesh and use it.  This command is `BED_MESH_CALIBRATE` called after `G28` (the homing override).  Now, here is what I do, I keep my start and end gcode sequences in the Klipper printer.cfg as `gcode_macro`s.  This allows me to use multiple slicers and keep my start and end gcode the same.  I have them mapped to `START_PRINT` and `END_PRINT`, then in any slicer i am using, I can just put those in and let Klipper handle the sequences.  I have included my printer.cfg at the bottom of this page so you take a look at my start up and end print gcode.

## Configure The Z Offset
So, for me this was the hardest part, I had the probe working, and at least stopping from crashing into the bed, but, I couldn't figure out how to actually make the printer know the space from the probe being triggered, to the bottom of the nozzle.  Which is obviously crucial for the hotend to know how much lower to go past the point where the BLTouch is triggered and be the perfect height above the bed to start printing.  Well, its actually really simple.  At least thats how I feel about it now that I know how to do it.  And after figuring it out, the only time I have had to re-calibrate my Z offset is when I replace the nozzle.

1. Go into your printer.cfg and change the `z_offset` to 10 in the `[bltouch]` section.  This number is completely arbitrary.  Just need a value there, and one that will be easy to do math with later.
2. Home the printer, `G28`.  as long as your `[bltouch]` settings are correct, the BLTouch pin should engage, and the Z axis should lower and stop, once its triggered.  I kept my finger on the power button at this point to stop the hotend from crashing into the bed if need be.
3. Next tell the printer to go into `Relative Position` mode: `G91`
4. Now move the hotend over the spot the BLTouch probed to home the bed: `G1 X-47`
5. Get a piece of paper and get ready to bed level like you always do, lower the Z axis by -.05 until the nozzle scrapes the paper.  You can do this with the display control, or I use [OctoPod](https://itunes.apple.com/us/app/octopod-for-octoprint/id1412557625?mt=8) on my iOS device.
6. Once the Z axis is where it needs to be, you can run `GET_POSISTION` in the terminal and then just do the math:
    * `[printer.cfg [bltouch] z_offset] - [z value from GET_POSITION] = z_offset`
    * IE: 10 - 8.5 = 1.5
7. Replace your the `z_offset` value in the `[bltouch]` section with your new calculated value.
8. Print and be happy as your first layer runs perfect!

I actually left my printer off for about 4 weeks recently, turned it on and fired off a print, first layer still layed perfect.  Cheers!

---

### Update
* 20190609 - A recent Klipper update moved the `samples: 2` and `sample_retract_distance: 8.0` from the `[bed_mesh]` section to the `[bltouch]` section.  I have updated this article to reflect those changes.

---

### My Ender3 Klipper printer.cfg

{% gist 88172c35ec1b6291ab7ad867a997c5a5 %}
