---
layout: post
title: PiLess BrewPi
category: homebrewing
description: Building a BrewPi without a Raspberry Pi using the BrewPiLess implementation.
tags: homebrewing raspberrypi 3dprinting
---

Its no secret that temperature control is a huge aspect of brewing beer successfully.  It can have dramatic side effects if you don't keep the temperature in the yeasts range, or you may not be able to achieve flavors that you're seeking from the yeast fermenting at a specific temperature.  Now, in my past life as @dotps1, I covered in detail how to make a BrewPi.  A temperature controller running with a Raspberry Pi and an Arduino board (that blog post can be found [here](https://dotps1.github.io/homebrewing/2018/01/22/brewing-your-own-brewpi.html), for now anyway, I'm not sure how long I will be keeping https://dotps1.github.io up).  The BrewPi is defiantly a solid temperature controller solution, that allows for many customization, and the ability to create temperature profiles, allowing for temperature ramp up and cold crashing.  And most importantly, it handles temperature swing.  Unfortunately, the BrewPi is no longer officially supported on the Raspberry Pi/Arduino configuration, and as you can see the branch hasn't been touched in years, https://github.com/BrewPi/brewpi-www/tree/legacy.  I made this clear in my last walk through, which is over a year old now, and I had to fork the project and fix a bunch of dependency issues, that probably are already out of date.

## Enter the PiLess BrewPi
So, theres another implementation of the BrewPi software out there, called [BrewPiLess](https://github.com/vitotai/BrewPiLess).  And the best part is, it doesn't need a Raspberry Pi, or even an Arduino Controller!  And the project is actively developed, which means new features, and bug fixes.  One of the really great features the BrewPiLess has, is that it supports logging data to [BrewFather](https://brewfather.com).  There you can attach the device to a batch, and keep track of the temperature as the batch is fermenting, and all the data is stored in one place. 

I was able to get this up and running, oh, I'd say about 10x faster then I got the BrewPi up.  However, if you read my last blog on makeing a BrewPi, I housed everything in an outdoor sprinkler control box, it was huge and there where power cords and temperature probes hanging all over the place.  Well this time, I wanted something fully modular, where all the cords where removable, and much much smaller.  As small as absolutely possible.  I reached my goal, but if you want your _BPL_ exactly the same as mine, you will need to solder some connections in this build, and have access to a 3D Printer for the enclosure.  These are not show stoppers, there is plenty of other ways to put everything together, this is just _my way_ based on the things I regretted doing when I built my BrewPi, and features I wanted in this build.
![Complete]({{ site.baseurl }}/images/2019-05-01-piless-brewpi/complete.jpg)

## Shopping List
_Most of this stuff comes in bulk, so find a buddy that wants one too, and split the cost!_
* [ESP8266 NodeMCU](https://www.amazon.com/gp/product/B01IK9GEQG/ref=as_li_ss_tl?&imprToken=nLUZlu1.uJ1qUr1jcx9QNQ&slotNum=0&ie=UTF8&psc=1&linkCode=ll1&tag=angrymrtom-20&linkId=ff61ed8286632b653d985f627eba863a&language=en_US) (you don't need two of these, but at that price, might as well, the second one is almost free)
* [Two Channel Relay Module](https://www.amazon.com/gp/product/B01MUATVXX/ref=as_li_ss_tl?&imprToken=nLUZlu1.uJ1qUr1jcx9QNQ&slotNum=1&ie=UTF8&psc=1&linkCode=ll1&tag=angrymrtom-20&linkId=2a9b300b40efe61b1e1152932375fff4&language=en_US)
* [20x4 LCD Module](https://www.amazon.com/gp/product/B01GPUMP9C/ref=as_li_ss_tl?&imprToken=nLUZlu1.uJ1qUr1jcx9QNQ&slotNum=2&ie=UTF8&psc=1&linkCode=ll1&tag=angrymrtom-20&linkId=3c5ac9a99e1f03a95b47641a4d335bfd&language=en_US) (this is optional if you would like a display on the controller)
* [AC->DC 5V PSU](https://www.amazon.com/gp/product/B07CBS768L/ref=as_li_ss_tl?&imprToken=nLUZlu1.uJ1qUr1jcx9QNQ&slotNum=3&ie=UTF8&psc=1&linkCode=ll1&tag=angrymrtom-20&linkId=d23b9dcedf01b4de748275e0335f6fba&language=en_US) (this acts a power supply unit inside the enclosure, powering all the componets, else you can power things with USB bricks, but that sounds like a mess)
* [Mini XLR Male connectors (2 needed)](https://www.amazon.com/gp/product/B014EJY0JE/ref=as_li_ss_tl?&imprToken=nLUZlu1.uJ1qUr1jcx9QNQ&slotNum=4&ie=UTF8&psc=1&linkCode=ll1&tag=angrymrtom-20&linkId=ddc6d8cbb0138f9d6cc78bbadb54d151&language=en_US) (these are optional allowing for the temperature probes to be disconnected from the enclosure)
* [Mini XLR Female connectors (2 needed)](https://www.amazon.com/gp/product/B01J33CF48/ref=as_li_ss_tl?&imprToken=nLUZlu1.uJ1qUr1jcx9QNQ&slotNum=5&ie=UTF8&psc=1&linkCode=ll1&tag=angrymrtom-20&linkId=b7b3577fdfe015049dd8c4b80a6d0deb&language=en_US) (these are optional allowing for the temperature probes to be disconnected from the enclosure)
* [AC 3-Pin Outlets (2 needed)](https://www.amazon.com/gp/product/B01M3URWIT/ref=as_li_ss_tl?&imprToken=nLUZlu1.uJ1qUr1jcx9QNQ&slotNum=6&ie=UTF8&psc=1&linkCode=ll1&tag=angrymrtom-20&linkId=2fca3cda54bc73b3e8f4a0194eda161b&language=en_US)
* [3-Position Screw Terminal](https://www.amazon.com/gp/product/B00X73S52M/ref=as_li_ss_tl?&imprToken=nLUZlu1.uJ1qUr1jcx9QNQ&slotNum=7&ie=UTF8&psc=1&linkCode=ll1&tag=angrymrtom-20&linkId=8e68886aa9d3960dd3a2da2eaf8c285f&language=en_US)
* [AC Rocker Switch](https://www.amazon.com/gp/product/B00NWO68JI/ref=as_li_ss_tl?&imprToken=nLUZlu1.uJ1qUr1jcx9QNQ&slotNum=8&ie=UTF8&psc=1&linkCode=ll1&tag=angrymrtom-20&linkId=46484c9900f8aeb9bc2abe0d1964eb1b&language=en_US)
* [DS18B20 Temperature Probes (2 needed)](https://www.amazon.com/gp/product/B00EU70ZL8/ref=as_li_ss_tl?&imprToken=nLUZlu1.uJ1qUr1jcx9QNQ&slotNum=9&ie=UTF8&psc=1&linkCode=ll1&tag=angrymrtom-20&linkId=ff3a58618c96d064dba3139843a7809c&language=en_US)
* [4.7k Resistor](https://www.amazon.com/Projects-25EP5144K70-4-7k-Resistors-Pack/dp/B0185FC5OK/ref=as_li_ss_tl?keywords=4.7k+resistor&qid=1556653428&s=gateway&sr=8-2-spons&psc=1&linkCode=ll1&tag=angrymrtom-20&linkId=021d72a211058cd98c5afe45fa38b360&language=en_US) (only 1 is needed, but good luck finding just 1 for sale)
* [Breadboard Jumpers](https://www.amazon.com/HiLetgo-Breadboard-Prototype-Assortment-Raspberry/dp/B077X7MKHN/ref=as_li_ss_tl?crid=3Q681FUGHK6V2&keywords=female+to+female+jumper+wires&qid=1556724289&s=gateway&sprefix=female+to+female+jum,aps,157&sr=8-6&linkCode=ll1&tag=angrymrtom-20&linkId=4474fd5023fb444fcc74105cd7115365&language=en_US) (really only need female jumpers for this project)
* Misc screws, wire, shrink tube, and wire nuts/connectors

## Pro Tip
I am going to suggest starting this project by printing the enclosure.  I am going to suggest this for a few reasons, first of all, most the customization I used in my build are only really needed if you are going to use my enclosure design.  Things like the plugs, XLR connections, AC Rocker Switch and the LCD are all going to work perfect in my enclosure,  Second, it takes a bit to print, and well, everything mounts in the enclosure, so you're going to kinda need that part right away.  If you're not going to use my design, you can probably skip this part, and just mock everything up so you can figure out how your going to house this contraption.

The enclosure I designed is available on [Thingiverse](https://www.thingiverse.com/thing:3042974).  It is one of the first things I've designed, so I'm sure there are things that _could be done better_ but it works well enough for what it is.  I'm not going to go into print settings for this as its out of the scope of this walk through.  So, if you're going this route, download the stl and get it printing because it will take a bit, you can print the lid while you are assembling the rest of the BPL.

## Time to make the chimi-f***ing-changas
So, the first thing I want to say about this system, is that its dealing with 110v AC power, you will be running that power to the AC->DC converter (the NodeMCU and LCD run on DC power), and to the relay module, which essentially turns on/off your power outlets, which will control your Heating and Cooling in your fermentation chamber.  I cannot over stress this enough, **IF YOU ARE NOT COMFORTABLE WORKING WITH ELECTRICITY OR DON'T KNOW HOW BASIC ELECTRICAL WIRING WORKS, STOP WHAT YOU ARE DOING, PUT YOUR TOOLS DOWN, AND FIND SOMEONE WHO DOES**!  Ok, with that being said, lets get down to business.

### Install the BrewPiLess bin on the ESP8266 NodeMCU
1. Download the BrewPiLess Repository:
    * `git clone https://github.com/vitotai/BrewPiLess.git`
2. If you don't have it installed, you will need to download and install [VSCode](https://code.visualstudio.com).
3. Open the repository in VSCode.  If you don't have [Platform.io](https://platformio.org/install/ide?install=vscode) installed, go to the extensions workspace and install it.
    * On my mac, it failed to install the first, and it was because `virtualenv` needed to be installed.  Completing the three steps listed in the _Perquisites_ section [here](http://docs.platformio.org/en/latest/installation.html#virtual-environment), and then uninstall/reinstall the extension fixed the issue.
4. Connect the NodeMCU to your computer via USB, and flash the bin to the board (make sure you are in the working directory of the repo):
    * `command+shit+p`
        * `>PlatformIO: Build`
        * `>PlatformIO: Upload`
        * `>PlatformIO: Clean`
5. After the flash is complete, the NodeMCU will begin broadcasting its own SSID, you need to connect to to this WiFi network to configure it to connect to your home WiFi.
    * Username: `brewpiless`
    * Password: `brewpiless`
    * Open a browser it should auto connect to a config page, where you can change the username, password, and the title.
    * Save the changes and on the next page enter you home wireless info.
    * After this is complete you can connect back to your home wifi.

### Soldering
There are a few things to solder, and some need to be done before assembly:
1. Solder your wires to the AC->DC 5V PSU, you won't be able to do this after its in the enclosure.
    * On the **POWER IN** side, you need 14-16 AWG wire, basically the same wire thats in the walls of your house, I clipped an old Molex computer cable and stole wire from that.  This wire needs to be heavier gauge because it will have the 110v that is coming from your house to it, then will convert that to 5v DC.
    * On the **POWER OUT** side, you need **THREE** Hot and Ground leads.  These will power the NodeMCU, LCD and Relay.  Just clip one end of the breadboard jumper wires for this.
    * you can either solder the wire right to the board, or there is screw terminals that you could get as well, that you would solder to the board, and then you can screw the wires into the terminals, the choice is yours.
2. The temperature probes need to be soldered to the female XLR connectors, the wire to pin layout don't matter really, just as long as you do both of them the same.
    * Unscrew the connector, put the plastic housing over the temperature probe wire, and solder the three wires from the probe to the three pin cups on the connector.
    * Repeat this step for the other temperature probe.
    * One thing I did was cut one probe a bit shorter then the other, for two reasons. First, the probe to monitor the fridge temperature didn't need to be 6' long.  Second, it makes it easier for me to keep track of which probe is for the chamber temperature and which is for the beer temperature.
3. While you have your solder gear out, might as well solder the wires to the male fittings as well.
    * Sacrifice a few jumper wires for this, I would recommend using the same colors as the wires that where in the temperature probes so you can keep every thing lined up. The connectors are keyed so they can only go together one way.
    * leave about 2-3 inches (cut a jumper in half) of wire, these will connect to the three position screw terminal that is directly behind the connectors in the enclosure (see the _layout_ image below).
    * Solder the wires to the pin cups, again, be sure your wire colors match up, your basically just making a pass thru connection in the enclosure, that is detachable if need be.

### Assembly
Wiring diagram:
![Wiring]({{ site.baseurl }}/images/2019-05-01-piless-brewpi/wiring.jpg)
Layout diagram (I wasn't able to find a 3D model of the AC-DC 5V PSU, that goes in with the Power IN facing way from the the LCD in the big empty space):
![Layout]({{ site.baseurl }}/images/2019-05-01-piless-brewpi/layout.png)
1. At this point, you can start putting all the pieces in the enclosure.
    * Place each board on its mounting posts, I used old computer motherboard screws to hold them down.
    * The three position screw terminal I glued to the post.
2. Use the diagram above to connect all the jumpers.
    * The power for the NodeMCU will be supplied from the AC->DC 5V PSU to the 5V pin on the board.
    * Connect a ground lead from the AC->DC 5V PSU to any ground on the NodeMCU board.
    * In the diagram it shows the 5V pin is used to power the relay module, but that is assuming the NodeMCU is being powered VIA the USB port.  Which is not the case, the 5V (maybe also labeled 'Vin') pin on the board will be used to power the NodeMCU itself, and then you can use one of the 3 power leads from the AC->DC 5V PSU to power the relay module.
    * Be sure to add the resistor in the three position screw terminal when wiring in the temperature probes.
3. Next wire in the power switch and the outlets.
    * AC Rocker switch wiring digram:
    ![Switch Wiring]({{ site.baseurl }}/images/2019-05-01-piless-brewpi/switch_wiring.jpg)
    * From the power switch you will have three leads, 1 to the AC->DC 5V PSU, and 1 for each outlet. I used lever action wire nuts for this connection because there is 4 14AWG wires connecting at this point.
    * The Relay acts as a switch to complete the circuit, so the hot goes in, and the back out to outlet, and the naturel goes directly to the outlet.

There really isn't much more I can say about the wiring of this, all the information needed is in the diagram, there is more then one way to skin a cat, so just make sure your wiring is following the diagram and you'll be fine.  One thing to note, my fridge that I have plugged in the cooling outlet must draw a lot of amps, because the power switch comes with a 5 amp fuse, and when it turned on it, blew that, I had to replace it with a 10 amp fuse.
![Front Wiring]({{ site.baseurl }}/images/2019-05-01-piless-brewpi/front_wiring.jpg)
![Rear Wiring]({{ site.baseurl }}/images/2019-05-01-piless-brewpi/rear_wiring.jpg)

## BrewPiLess Configuration
Once everything is up an running, there isn't much left to do.  Just need to set which plug is the hot and cold, and which probes are the chamber, and the beer.
1. Connect to your BrewPiLess via a web browser at http://brewpiless.local (or whatever you named it in the setup screen.)
2. Go to the `Device Setup` screen on the top right corner.
    * Be sure to click the `Erase EEPROM` button at the top to be sure everything is reset.
    * Click `Refresh Device List` on the top load all your devices.
    * You have to manually select a `Device` slot for each probe and plug, but these are arbitrary, they just have to be unique.
    * The temperature probes will have an `Address` value.
    * Setup 4 devices:
        * Chamber Temp
        * Beer Temp
        * Chamber Heater
        * Chamber Cooler
    * I am not sure if there is a better way to then trial and error to identify which is which, for the probes you can just hold in your hand and the value will change on the display.  For the plugs you will need to actually set a `Beer` or `Fridge` temperature value and then one or the other will turn on.
3. Once your get everything setup, I would recommend backing up the configuration from the `Device Setup` screen incase you ever need to reload or rebuild, then you won't have to play the guessing game on which device is which.

## Extra Credit
If your like me, you are going to want to change the temperature units from C to F, because the NodeMCU has such little amount of space, there is no room for webpages to handle customization.  Instead, you do it with commands:
1. Navigate to http://brewpiless.local/testcmd.htm
2. Issue this command:
    * `j{"tempFormat":"F"}`
3. If your getting in the Kveik craze, and want to ferment hot AF, the you need to increase the `tempSetMax` value:
    * `j{"tempSetMax":"90.0"}`

There is plenty of customizing you can do, just check out the BrewPiLess wiki: https://github.com/vitotai/BrewPiLess/wiki

## Time for a Cold One
Whew, you did it!  Nice work!  Go pour yourself a cold one, I know I'm going too, sorry if I missed anything, this was a pretty intense walk through.  Cheers!
