---
layout: post
title: "Wifi Woes resolved with Ubiquiti"
---

## Fast Wifi is possible!

I have declared my Ubiquiti experiment a success! I am consistently measuring internet speeds over 100 MB/s with the lowest I've seen around 40 MB/s.  This is a huge upgrade from seeing 5-8 MB/s regularly with the highest being around 15 MB/s (see [my previous post][1] for more details on the diagnosis of my old system).

In this post, I want to document my installatin process because it was a bit of a doozy.  I also want to offer some recommendations in case anyone else is considering investing in a Ubiquiti wifi set up.

## The Installation Process (A Network Bootstrapping Wrestling Match)

First, here is the hardware I ordered [^1]:
* [Unifi Security Gateway (USG)][USG]
* [Unifi AP AC Lite][AP]


The first step after plugging everything in and powering it on was "adopting" the devices to a controller.  I started out putting the controller software on my laptop for expediency with the eventual goal of running it on a RaspberryPi.

### Laptop Controller 

My laptop does not have an ethernet port, so I connected to the AP using the QR code on bottom side of the AP.  I had to use the iOS App to do this scanning because the Android app wasn't working for me.  This gave me the SSID for a hidden network (the MAC address of the AP w/o the colons) and a password.  I was able to then see the USG in the controller interface, but I could not see the AP.  I assumed this was because I was connected through the AP and that in order for the controller to adopt the AP it had to be running independently of it[^2].

### Raspberry Pi Controller over Ethernet

This was when I decided to plug the Raspberry Pi directly in to the USG andstart using it as the controller. I followed [a guide][rpi-unifi] to get the software installed and after a couple of speedbumps[^3], got the controller up and running. I was SSH'ing to the machine via my laptop which was still connected to the AP's MAC address network, and I assumed I needed to not be connected that way in order for the AP to be adoptable (an assumption that may have been flawed).  In order to work around that, I set up the RaspberryPi as [a wireless access point](https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md) and then connected and SSH'd that way.  

### Raspberry Pi Controller over WiFi

I then was able to adopt the AP and the USG, and things were looking good.  However, I needed to free up an ethernet port in order to plug in the brigde for my Hue lights, so I wanted to get my Raspberry Pi acting as the controller over Wifi.  I figured this would be a simple exercise of connecting the Raspberry Pi to Wifi and updating some configuration settings.  Once I did this, and unplugged the ethernet cable, it started reporting the USG as not connected.  Apparently, the USG wants the controller to be on the same subnet as it, and in this case the AP (and therefore the controller) was on the LAN2 subnet[^4].  I tried naively swapping the AP to the LAN1 port, but this really messed evertyhing up, so I factory reset everything and started from the beginning.

On my next attempt, after I got the Raspberry Pi up and running as the controller. I unplugged it, and plugged the AP into the same port.  This way the AP was running on the LAN1 subnet.  After reconnecting the Raspberry pi, everything worked, and I recorded much faster speeds than I had previously read.

In the future, I want to look into running a Controller from an EC2 or GCE instance so I don't have to keep my RaspberryPi plugged in to power all the time.


## Reccomendations

* Be generous with your ethernet ports - investing in an ethernet switch will save you heartache even if you only use it for the initial set up.
* Buying a "Cloud Key" will also make your setup life a lot easier, some of the "Discovery" and "Set Up" tools have special features that only work with a Cloud Key.
* Be sure your Controller is on the same subnet as the USG.
* If your modem doesn't immediately work with the USG, unplug everything, wait a few minutes and try again.  (This may seem like the technology equivalent of a ceremonial sacrifice, but it worked for me, and I'm willing to chalk it up to something in the modem needing to be cleared out from the previous router)

## Take-away

Hopefully this saves someone from the mostly self-inflicted torment I experienced.  Overall, I am very happy with the Unifi set up and would recommend it to anyone who's willing to invest a little bit extra in the set up.



[1]:{% post_url 2018-01-04-wifi-debug %}

[USG]: https://www.amazon.com/Ubiquiti-Unifi-Security-Gateway-USG/dp/B00LV8YZLK/ref=sr_1_1?ie=UTF8&qid=1515081742&sr=8-1&keywords=ubiquiti+usg+unifi+security+gateway

[AP]: https://www.amazon.com/dp/B016K4GQVG/ref=sr_ob_1?ie=UTF8&qid=1515081752&sr=8-1

[rpi-unifi]: http://www.technologist.site/2016/06/02/how-to-install-ubiquiti-unifi-controller-5-on-the-raspberry-pi/


[^1]: If you've read up on the Unifi line of products, you may notice this is missing a "Controller".  I have RaspberryPi laying around which I decided I was going to use at the Controller, but more on that later.

[^2]: Looking back, there was probably a way to manually adopt the AP by SSH'ing to it as described in this [forum post](https://community.ubnt.com/t5/UniFi-Wireless/Changing-controller-IP/td-p/255168), but I did not know enough at the time to try that.

[^3]: First, I had some weird version of Raspian installed which made it impossible to install the right packages.  I had to [flash a new image](https://www.raspberrypi.org/documentation/installation/installing-images/) and remember to [`touch ssh`](https://www.raspberrypi.org/documentation/remote-access/ssh/) in the boot directory in order to enable SSH access on a fresh install because I didn't have a monitor or keyboard with me.

[^4]: I gathered this from [an article](https://help.ubnt.com/hc/en-us/articles/236281367-UniFi-How-to-Adopt-a-USG-into-an-Existing-Network) about how to adopt a USG into an existing network.  It says "If the controller is on a subnet other than the USG’s default 192.168.1.0/24, it is necessary to change the USG’s LAN IP so the controller and USG can communicate".
