---
layout: post
title:  ""Installing Linux on a MacBook Air"
date:   2016-05-09 08:21:54 -0700
---

See my last post about the saga involved in getting my laptop up and running again.

I decided I wanted to start running linux on my laptop.  I previously had a desktop running linux that I would code on from time to time, but currently I don't have it set up, so I was seeking a different source of linux in my life.

I did a factory reset of my laptop, and it was kind of scary how old it felt. I bought it 5 years ago, but somehow that didn't feel like a long time.  When I installed chrome on it, it warned that my OS was no longer supported! I was in that statistic of old OS's you see crop up on Mixpanel/Google Analytics and wonder how somebody is still running that OS.

I settled on Debian as my distribution.  I have run a Fedora and Ubuntu before. They both seem fine.  The main impetus was that a friend of mine had recently set up a linux machine and used Debian after wrestling with Arch Linux for a few weeks.  I also found some guides that indicated that installing Debian was doable on a macbook air.

I downloaded the Debian amd64 DVD .iso files from [here] (The "amd" thing threw me for a loop since the processor in this machine is made by intel, but never fear, "amd64" simply indicates a 64 bit operating system (citation needed)).  I didn't have a torrent client, so I went ahead and used uTorrent.

I then converted the .iso files to a .dmg file by first 'cat'ing them together (I was a little surprised that worked, I guess I don't know much about the .iso format).  I then followed the instructions [here] to make a bootable USB stick out of it.

One speedbump I hit was that Macbook's have a wireless chip made by broadcom.  Broadcom does not opensource the drivers for these chips, so Debian does not include them in their .iso's by default.  I came across this as I was installing the OS and it asked me to setup network settings.  It gives you the opportunity to provide them during install, but I only had one USB stick handy, so I pressed on without. (I'm not sure which specific file it would have wanted here.)

The install went smoothly aside from the wireless chip issue.  I then had the task of getting the broadcom drivers on to my lonely wifi-less machine. I found that the "wl" module was the recommended way of getting the broadcom drivers via this link.  I was browsing around some discussions and found this discussion of only needing a single .deb file to get rolling.  I found that archive [here] and noted that it had a dependency on *dep*.  This was not installed on my machine, but "apt" knew about it.  I figured out this was because it is included in the install disk, but not installed by default.  

I discovered that my apt configuration was looking for a cd-rom when I tried to install it.  After poking around, I found that you could re-route this to a folder which I pointed to the USB stick with the install files on it. *show example config file*.  This let me install wireless-tools *check name*.

I plugged my usb stick back into my internet connected computer.  I reformatted it and put the .deb file for the *package name* there.  *note usb stick file format*.  I ran the install commands *list commands* and then the commands on the wl page *link* and was able to connect to my network!

My next step was to install Chrome.  To do this, I downloaded the .deb file and ran *install commands*.  This gave me a strange error message regarding lib-pango.  I changed the sources of my apt installation again, this time basing it off the list *here* and included the "non-free" versions.  After doing this, I installed chrome without an issue.

I also needed to add myself to the sudo group so that I could run `sudo apt-get install`.  The commands for that are *sudo commands*.

The next thing I did was follow the helpful trackpad tips listed *here*.  The palm detection was huge as well as the tap to click.

And now I have a working macbook air running Debian!
