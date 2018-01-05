---
layout: post
title: "Debugging Slow Wifi"
---


# Introduction

I ran into the ever common problem of slow WiFi recently.  In this post I want to detail the experiments I ran and what I gleaned from the results.


# Setting the Stage

First I'll describe the setup I have for my home wifi network and the problems I was experiencing.

Symptoms:
* No problem streaming Netflix
* Trouble loading web pages occasionally

I had a weird set of problems in that I typically could stream episodes of shows on Netflix without any huge problems, but sometimes loading sites would be unbearably slow.  It was common enough to be a huge pain, but the times when it worked well confused me.  Why was it only sometimes slow?


Hardware:
* Modem: [Arris Surfboard SB6141](https://www.amazon.com/gp/product/B00AJHDZSI/ref=oh_aui_search_detailpage?ie=UTF8&psc=1)
* Router/AP: [Google OnHub](https://on.google.com/hub/features/) by [TP-Link](http://www.tp-link.com/us/products/details/cat-9_TGR1900.html)

# Investigation

I wanted to check if my internet was actually slow or whether I was just impatient.  I ran speedtests from several different providers, and it was in fact slow. (billed speed was 100 Mb/s, observed was between 3-8 Mb/s).

At first, I was convinced that Comcast was to blame for my slow wifi speed.
However, I ruled this out by checking the internet speed over ethernet.  Comcast was delivering speeds to my router that were much higher than I was observing at my laptop. (observed ethernet speeds between 56-75 Mb/s)

Next, I wanted to establish a baseline for what the point to point speed on my network was.  This came out around 5 Mb/s which was really slow!

Next, I thought maybe there was a lot of WiFi congestion in my area.  Maybe my router was picking a congested channel, or there was some kind of crazy interference in my apartment.  To test this, I ran my raspberry pi right next to my router in AP mode.  I was very surprised when the Raspberry Pi was able to get 33 Mb/s running on the same channel as my router.



# Experiments


## Speed Tests
Ran speed tests on 3 different sites: [fast.com](http://fast.com) (Netflix's Speed test site), [speedof.me](http://speedof.me) (Javascript based graphs and reports), and [speedtest.net](http://speedtest.net).  


Results:

- Fast.com: 8 Mbps
- speedtest.net: 4.6 Mbps
- speedof.me: 3.15 Mbps


## Raspberry Pi on Ethernet

Setup:

I plugged my Raspberry Pi into the ethernet port of my router.  I then downloaded [speedtest-cli](https://github.com/sivel/speedtest-cli) (a CLI for running speed tests against speedtest.net).  I also ran a test with `wget`[^1] 

Results:
- speedtest-cli: 75 Mb/s
- wget: 6.9 MB/s => 56 Mb/s


## Laptop to Laptop communication

Setup:
I installed [`iperf`](https://iperf.fr/iperf-download.php) on 2 laptops. Laptop 1 is running Linux on a 2011 Macbook Air and was running `iperf -s` (the server).  Laptop 2 is a 2017 Macbook Pro running `iperf -c`.  I started out running with the default parameters which runs a test for 10 seconds, then switched to longer times using `iperf -c -t 60` to run the test for 60 seconds.

Results:

- Average for 10s (6 trials): 17.7 Mb/s
- Average for 60s (3 trials): 5 Mb/s


## Raspberry Pi as AP

Setup:
I put my raspberry pi in AP mode[^2] and had my laptop connect to it directly.

Results:
- `iperf -c -t 60` : 33.4 Mb/s


# Conclusions

My conclusion is that my router is not performing.  I tried power cycling it to see if that would help, but the measurements were the same after.  I am not certain why this is the case, especially since it is supposed to have 13 antennae which I would expect to lead to high LAN bandwidth.  My next experiment is to order some [UniFi](https://unifi-sdn.ubnt.com/) hardware and see if was that improves the situation[^3].

I never got a good answer as to why I was able to stream Netflix but regular websites would have trouble loading.

Hopefully my new hardware does the trick. I plan on writing a follow up post when I have it all set up to see if it makes a difference.



[^1]: The exact command I ran:
`wget --output-document=/dev/null http://speedtest.wdc01.softlayer.com/downloads/test500.zip` NB: `wget` reports speeds in megaBYTES not megaBITS (like most wifi tools).  Multiply by 8 to get a comparable number.

[^2]: To setup a Raspberry Pi as an AP, I used `hostapd` and `udhcpd` which you can find instructions for on [raspberrypi.org](https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md)


[^3]: I am going with a [USG](https://www.amazon.com/Ubiquiti-Unifi-Security-Gateway-USG/dp/B00LV8YZLK/ref=sr_1_1?ie=UTF8&qid=1515081742&sr=8-1&keywords=ubiquiti+usg+unifi+security+gateway) and a [UniFi AP AC Lite](https://www.amazon.com/dp/B016K4GQVG/ref=sr_ob_1?ie=UTF8&qid=1515081752&sr=8-1).  I went with Ubiquiti based on recommendations from some coworkers and a blog post by [Troy Hunt](https://www.troyhunt.com/ubiquiti-all-the-things-how-i-finally-fixed-my-dodgy-wifi/) that describes similar frustrations as mine.  I debated whether to get an EdgeMax router over the USG after seeing some discussion in Troy's [gist](https://gist.github.com/troyhunt/86ce1de40e58b1eed0961ce6a7a906d5).  I ended up going with the USG since I don't believe I'll benefit from the more advanced configuration opions of the EdgeMax, and I like the idea of all of it working with the UniFi portal.
