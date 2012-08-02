---
layout: post
author: dagingaa
title: Playing around with the Raspberry Pi
category: Technology
---
_This post was originally posted on [Comoyo Engineering](http://comoyo.github.com/blog/2012/07/23/playing-with-the-raspberry-pi/)._

At work this friday, we finally got our hands on the [Raspberry Pi](http://www.raspberrypi.org/). I got to spend some hours with the device, and these are my thoughts. 

#First impression#
The Pi is small. Very small actually. The fact that this thing can do 1080p video, run Debian and Arch Linux, and many other cool tricks, for little over 200 NOK, baffles my mind. Only 4 years ago I built my HTPC for 5000 NOK in a box the size of a stereo receiver, and this thing can handle hi def media a lot better, in a package much, much smaller. 

![Raspberry Pi](/assets/img/posts/raspberry/raspberry.jpg)

The magic is of course in the [Broadcom BCM2835](http://www.broadcom.com/products/BCM2835) and the excellent hardware design done by the Raspberry folks. The CPU is an ARM1176JZ-F Applications Processor, and the GPU a VideoCore IV. The manufacturer claims it is capable of playing back 1080p H.264 at 40Mb/s. However, the CPU is only 700 MHz, and seems to be the real bottleneck of the system. This is especially visible when trying to run a browser using the recommended linux install, Raspbian.

#Installing Raspbian#
To install or do anything with your new Pi, you first have to get a SD card. Luckily, I was prepared, and had one handy. The first thing most Raspberry Pi owners will try is the official Debian for Raspberry build, called [Raspbian](http://www.raspbian.org). This is an optimized build and has over 35,000 packages optimized for the floating point architecture of the Broadcom chip. 

Installing the Raspbian image on our Pi using Mac OS X was not something I’d call mass-market user friendly just yet. This may be a lot different on other operating systems though. We followed a [handy guide over at elinux.org](http://elinux.org/RPi_Easy_SD_Card_Setup#Copying_an_image_to_the_SD_Card_in_Mac_OS_X), and we will repeat the steps here.

First, we have to download and unzip the image itself. You can always find the latest image on http://www.raspberrypi.org/downloads. 

Now, we need to prepare our SD card.

Find the path of the SD card on your system.

    $ df -h
    Filesystem      Size   Used  Avail Capacity  Mounted on
    /dev/disk9s1   7.4Gi  2.6Mi  7.4Gi     1%    /Volumes/NO NAME
    
Unmount the drive

    $ diskutil unmount /dev/disk9s1
    Volume NO NAME on disk9s1 unmounted
    
The diskname for my raw disk is now /dev/rdisk9. Be in the directory where you extracted your image file and do the following command. This takes a while, so be patient. The only output is at the end.

    $ sudo dd bs=1m if=IMAGE_FILE_PATH.img of=/dev/rdisk9
    1850+0 records in
    1850+0 records out
    1939865600 bytes transferred in 419.827777 secs (4620622 bytes/sec) 
    
Eject the disk

    $ diskutil eject /dev/rdisk9

#Booting it up#
The hooked up Pi is by no means worthy of [/r/cableporn](http://www.reddit.com/r/cableporn), but I guess that isn’t the point. 

![Raspberry Pi hooked up](/assets/img/posts/raspberry/plugged_in.jpg)

The booting itself is fairly quick, taking only 20 seconds or so. At first boot, we are presented with the usual blue Debian installer, which lets us configure the pi to our liking. A quick reboot later and we’re inside a desktop manager unknown to me.

![Raspbian desktop](/assets/img/posts/raspberry/raspbian.jpg)

The experience is surprisingly fast considering this is a 200 NOK device, but not something I would ever use for a regular desktop experience except for emergencies. The browser is very sluggish, killing my hopes for a custom web interface made for TVs running on this thing. It does however, come preinstalled with Scratch and Python, which I guess is the educational part of the goal for the Raspberry team. I wanted to try XBMC on this thing, so I didn’t do much testing in forms of benchmarking on the Raspbian image. 

#Installing Raspbmc#
[Raspbmc](http://www.raspbmc.com/) is a simple and lightweight media center distribution for the Raspberry Pi, created by Sam Nazarko. It is basicly [XBMC](http://www.xbmc.org) compiled for the Raspberry Pi.

Installation using OS X is fairly simple.

    curl -O http://svn.stmlabs.com/svn/raspbmc/testing/installers/python/install.py
    chmod +x install.py
    sudo python install.py

And that’s it! Follow the on screen instructions, and make sure you select the correct disk!

#Booting Raspbmc#
First time boot requires the Pi to connect to an update server. This process takes about 15 minutes. After that, we are greeted with the usual XBMC splash screen. 

![Raspbmc splash](/assets/img/posts/raspberry/raspbmc.jpg)

Initial impressions are fairly meager however. The Pi is unresponsive for some time, and the UI feels sluggish. Installing add-ons was painful. However, a quick reboot fixed most of these problems, probably because XBMC did some initial checks of outdated addons in the background. 

To test the performance of the device, we first tried [The Dark Knight 1080p trailer](http://www.youtube.com/watch?v=yQ5U8suTUw0). It played back flawlessly, with little buffering needed. Instant playback. Pretty great experience.

![Raspberry Pi](/assets/img/posts/raspberry/dark_knight.jpg)

We also tried some movie clips we had laying around, both from a usb drive and over the network, and we had no issues what so ever with buffering or stuttering during playback.

#Closing thoughts#
The Pi, having just ramped up production and opened the floodgates to the masses, is a very exciting device. Already we see improvements and new products popping up trying to mimic and improve the idea put forth by the Pi. One such device on the way to our headquarters is the [Gooseberry](http://gooseberry.atspace.co.uk/), and many others are being released in the next six months. We look forward to the future of computing, and we will continue to look into the possibilities these new devices give us. 

Stay tuned!
