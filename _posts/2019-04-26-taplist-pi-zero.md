---
layout: post
title: Taplist Pi Zero
category: homebrewing
description: A walk through for creating a digital taplist with Taplist.io and Raspberry Pi Zero.
tags: homebrewing raspberrypi keezer
---

So, if in your homebrew life, you've moved on past bottling (because its it f***ing terrible) and are kegging your ferments, you've probably already come up with some way to label or mark your taps so you know what tap is pouring what beverage.  I know there is _a lot_ of articles out there on this topic, but, I wanted to share my experience on the matter, and hopefully, it can save you some time, and money.

As you can probably tell, I'm a bit of a tech nerd, so obviously I wanted digital signage.  I mean there is several options out there to do this manually.  Tap handles with black board surfaces you can write on with chalk, or even black board spray paint, you could paint on your kegerator or keezer.  Or maybe make some other type of labels that hang on the taps or whatever.  But I didn't want anything like that, I wanted an actual taplist.  One of the main reasons this was even remotely cost feasible for me, is I had an extra 22" LCD monitor laying around.  If you have to buy a monitor to pull this off, then this walk through probably isn't for you, but if not here is a list of things you'll need:

* [Raspberry Pi Zero](https://www.amazon.com/gp/product/B01L3IU6XS/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=angrymrtom-20&creative=9325&linkCode=as2&creativeASIN=B01L3IU6XS&linkId=5ad8ce250e1079a1dbff8a6f1cba3d0d)
* [MicroSD Card (8GB minimum)](https://www.amazon.com/gp/product/B0012Y2LLE/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=angrymrtom-20&creative=9325&linkCode=as2&creativeASIN=B0012Y2LLE&linkId=ee9d9d3e68054a23ab63d1f02537e85e)
* [1.2A Minimum Power Supply (basically any old cell phone charger)](https://www.amazon.com/gp/product/B00GF9T3I0/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=angrymrtom-20&creative=9325&linkCode=as2&creativeASIN=B00GF9T3I0&linkId=0d85b67b54f9a9ee66e63ae36f2a1b7c)
* [HDMI Cable (mini HDMI to what ever interface is on the display)](https://www.amazon.com/gp/product/B01KRKO4MM/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=angrymrtom-20&creative=9325&linkCode=as2&creativeASIN=B01KRKO4MM&linkId=a1915b0866bd47fda46a792c390f5334)
* [Raspberry Pi Zero Case (optional, you could also 3D print this too)](https://www.amazon.com/gp/product/B01HP636I4/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=angrymrtom-20&creative=9325&linkCode=as2&creativeASIN=B01HP636I4&linkId=ab3467b15878cbe5c1780d9c252cd551)

The first time I did this, I used a Raspberry Pi 3, which is probably over kill for hardware to make a taplist, especially when all we are going to do is display a web page.  A few months ago, I need a Raspberry Pi for another project, and when I looked to buy one, they are like $30, however, at the time, I found a Raspberry Pi Zero for $5 _(at the time of this writing it is ~$15)_.  It doesn't have near the hardware resources, but again, we are just going to be displaying a webpage.  _(Also, with taplist.io, you can use a kindle fire stick, I haven't ever set it up with that, but I do know that there instructions on the product page on how to set one up.)_  So, lets get started.

## taplist.io
The premise of this digital taplist, uses a free product (for personal use) called [Taplist.io](https://taplist.io).  You will need to create an account, and from there, you can create taps, build your beverage database, create kegs, attach kegs to taps.  I'm not going to go into very much detail on this part because there is so many variables, and so much customization you can do with it.  what ever you customize under the "Look And Feel" section of your dashboard, is what will be displayed on the final product when you are finished.  The other nice part is you get an online taplist you can share with anyone (you can find mine at the bottom of this page, by clicking the "beer mug" icon).

## Setting up the Raspberry Pi Zero
For the first part of this build, we need to install and configure the Raspberry Pi Zero, we will be installing the latest version of [Raspbian Stretch Lite](https://www.raspbian.org).
1. Download the latest img for **Raspbian Stretch Lite** [here](https://www.raspberrypi.org/downloads/raspbian/).
2. Open the `Disk Utility` app:
    * Press `CMD+SPACE` to open the `Spotlight` application.
    * In the search bar type `Disk Utility`.
3. In the top left had corner, select `View`, and be sure `Show All Devices` is selected.
4. Next, you will need to be sure the SD card is formated with the [FAT](https://en.wikipedia.org/wiki/Design_of_the_FAT_file_system) file system.
    * Select the SD drive in the left pane.
    * Select `Erase` from the top.
    * Make sure you pick `FAT (MS-DOS)` for the file system, and `MBR (Master Boot Record)` for the partition.
    * **THIS WILL ERASE THE ENTIRE SD CARD, SO BE SURE YOU HAVE SAVED ANYTHING YOU WANT TO EVER SEE AGAIN!**
5. Open a `terminal` using the `Spotlight` tool (see step 2) and type `Terminal`.
6. In the `terminal` application, find your disk number by running `diskutil list`
7. Note your disk number, it will be in the format of `/dev/disk2`, it may not actually be the number `2`, just look at the size of the disk and make sure its the correct one.
8. Unmount the disk with `diskutil unmountDisk /dev/disk2`, again, replace `2` with number of your SD card.
9. write the image to the SD Card with `dd`:
    * `sudo dd bs=1m if=Downloads/raspbian-stretch-lite.img of=/dev/rdisk4 conv=sync`
    * Replace `Downloads/raspbian-stretch-lite.img` with the path to the `.img` file you downloaded in step 1.
    * Again, replace `rdisk4` with the disk number you found in step 7.
    * This may take some time, took my system about 10 minutes on an 8GB SD Card with a Thunderbolt SD Card Reader.

## Configuring Raspbian Lite Stretch
After the img is done writing to the SD card, its time to being configuring Raspbian.  Because the Raspberry Pi Zero has such little hardware, we are going to try to keep this as minimal as possible.
1. SSH is disabled by default, so to enable it, you need to create an empty file called `ssh`:
    * `touch /Volumes/boot/ssh`
2. Next, you need to append `dtoverlay=dwc2` to the end of `/Volumes/boot/config.txt`.  You can do this either with `textEdit` or with a terminal editor.
3. This next step is a little tricky, first open `/Volumes/boot/cmdline.txt`, and located the text `rootwait`, immediately after that, add a space, and then this text: `modules-load=dwc2,g_ether`.  So it should look like this:
    * `rootwait modules-load=dwc2,g_ether`
4. Now go ahead and eject the SD card and then load it into the Raspberry Pi Zero and plug in the power.  It will take about 60 seconds to boot up, then connect it to your computer.
5. Next, go to `System Preferences > Network` and you will see a new device called `RNDIS/Ethernet Gadget`.  ![Network]({{ site.baseurl }}/images/2019-04-26-taplist-pi-zero/network.png)  This is the Raspberry Pi Zero, you can share your internet connection to it, which will then all you to SSH to the device.
6. Go to `System Preferences > Sharing` and enable internet sharing for the `RNDIS/Ethernet Gadget`. ![Sharing]({{ site.baseurl }}/images/2019-04-26-taplist-pi-zero/sharing.png)
7. You should now beable to `ssh` to the Raspberry Pi Zero:
    * `ssh pi@raspberrypi.local`
    * Default password is `raspberry`.
    * Type `yes` to accept the certificate thumbprint.
8. After you are connected, there are a few things that need to be set, run `raspi-config` and update the following:
    * **CHANGE THE DEFAULT PASSWORD** else anyone can now `ssh` to your taplist board with the default credentials!
    * Under `Boot Options` select `Desktop/CLI` and then select `Console Autologin`.
    * Lastly configure the wifi.
9. You are now connected to your wifi, and can update the system:
    * `sudo apt-get update -y`
    * `sudo apt-get upgrade -y`
    * `sudo apt-get dist-upgrade -y`
10. Now, install the minimum X Server Environment for Chromium:
    * `sudo apt-get install --no-install-recommends xserver-xorg x11-xserver-utils xinit openbox`
    * `sudo apt-get install --no-install-recommends chromium-browser`
11. You need to configure the auto start commands, and to do this, you will have to use a terminal based editor, like `nano` or `vi`.  I prefer `nano`:
    * `sudo nano /etc/xdg/openbox/autostart`
12. Replace the entire contents of the file with this:

    ```
    # Disable any form of screen saver / screen blanking / power management
    xset s off
    xset s noblank
    xset -dpms

    # Allow quitting the X server with CTRL-ATL-Backspace
    setxkbmap -option terminate:ctrl_alt_bksp

    # Start Chromium in kiosk mode
    sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' ~/.config/chromium/'Local State'
    sed -i 's/"exited_cleanly":false/"exited_cleanly":true/; s/"exit_type":"[^"]\+"/"exit_type":"Normal"/' ~/.config/chromium/Default/Preferences
    chromium-browser --disable-infobars --kiosk 'https://www.taplist.io/display'
    ```

13. You need to configure X Server to autostart and boot and load the display web page.
    * `nano ~/.bash_profile`
    * Add this to the file: `[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && startx -- -nocursor`
    * Save and close the file.

14. Lastly shut you Raspberry Pi Zero down `sudo shutdown now` and go attach it to your monitor.  You do not need to attach any peripherals, on its next boot, it will autologin as the `Pi` user, and start up `Chromium` and navigate to https://www.taplist.io/display.  The very first time this boots, there will be a code on the screen that you need to go to https://taplist.io/activate and enter in.

WHEW!  Thats it, hopefully you made it this far, and ended up with something like this ![Taplist]({{ site.baseurl }}/images/2019-04-26-taplist-pi-zero/taplist.jpg)

Cheers!

---

## Sources
* https://brandonb.ca/raspberry-pi-zero-w-headless-setup-on-macos
* https://bdking71.wordpress.com/2018/11/06/setup-an-information-kiosk-using-a-raspberry-pi-zero-w/
* https://taplist.io/help/raspberry-pi-setup
