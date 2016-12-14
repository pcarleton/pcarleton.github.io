---
layout: post
title: 'Doorbell Hacking Part 2: Buying Components'
---

In [Doorbell Hacking Part 1][1], my friend and I gained an understanding of how my doorbell system works.  We also came up with a plan to hook it up to my RaspberryPi given a few extra components.

In this post, I'll look at shopping for the electronic components I will need.

## Goals

It's worth noting the 2 goals of hooking up my doorbell in order of priority:

1. Buzz someone in from my phone
2. Get a notification if someone rings my doorbell on my phone

Goal #1 is mostly for being super lazy, or potentially hooking it up to a home assistant so I could say "Ok Google/Alexa -- Buzz them in".  Goal #2 is for some hypothetical situation when I wanted to let someone in remotely, maybe to drop off a package in the lobby.  #2 is not useful without #1. (For instance, getting notified that someone is pushing my doorbell is not very helpful if I do not have the ability to let them in.  Additionally, if I am home, I can hear the bell, so getting the notification is not necessary.)

## First Stop: The Relay

The first component I'll be looking at is the mechanical relay.  The relay will be what allows my RaspberryPi to stand in for the interior button to close the doorbell's circuit, activating the buzzer.  I didn't know much about relays, but I found [a nice rundown][2] on "explainthatstuff".  The gist is that the relay allows the small current output by the RaspberryPi to control the very large current of the doorbell system.

Another great post I found was by [National Instruments][3].  It explains several different types of relays: mechanical, reed, solid state, and FET (Field Effect Transistor).  The tl;dr is that mechanical relays are simple, but large and slow.  Reed relays are smaller and faster, but are susceptible to arc damage.  Solid state is faster and smaller, but does not provide complete isolation between the contacts since it is a MOSFET (metal–oxide–semiconductor field-effect transistor) doing the switching (as opposed to the mechanical and reed which keep the contacts physically separate.)  Finally a FET switch is the fastest, but it can only handle low-voltage scenarios.

After reading this, it is between a mechanical relay and a SSR (solid state relay).  I ruled the "Reed" relay out since I was seeing sparks on the hammer-switch, so I can't guarantee there won't be arcing in this component which means there is a risk the contacts will weld together, and my door will be permanently buzzed.  I also determined that I want a "non-latching" mechanical relay.  A latching relay leaves the connection in place after being activated even if the component loses power because it uses actual magnets to do the switching.  Again, I want to reduce the risk of continuously buzzing my door.

# Wait, what about an Optocoupler?

You may recall from [part 1][1] that the [project linked to by Twilio][4] suggested using an optocoupler.  Maybe it is because I like the word, or the fact that the component itself seems really clever, but I keep wondering if an opto-coupler is the way to go.

Reading about the SSR, it looks like a very similar component to an optocoupler.  Both use an LED to signal across isolated circuits, so I looked into why to pick one over the other.  I found a post explaining precisely that [here][5] by an electronics company called Renesas (checkout there homepage for some pretty hilarious marketing images).  The key take-aways were:

1. Optocouplers can only do DC (not AC)
2. Optocouplers are faster than SSR's
3. Optocouplers allow current through gradually based on brightness of LED, whereas SSR's are more ON/OFF
4. SSR's can be default on or default off whereas optocouplers are default-off.

The first item on that list seems like a deal-breaker for me.  If I can't operate an AC current, I can't very well buzz my door with it.  However, I found the datasheet for the optocoupler [here][6], and it lists the Maximum Cell Voltage (as opposed to the LED voltage) as 60V DC or AC.  This leads me to believe that the optocoupler can handle the AC voltage of my system which is only 22V AC.

With that one deemed False, the other 3 don't seem like clear advantages.  I asked [the author (@imightbeamy)][10] of the original Twilio post if there was a reason to pick one over the other.  She stressed that isolation was her main concern since she didn't want her RaspberryPi to get fried, but that any kind of relay should work.

## We'll do it live!

I decided to go ahead and buy all 3 components: a mechanical relay, an SSR and an optocoupler.  The total for my order was about $15, so I figured it was worth it to get all 3 and see if I could get each of them working.  Then I could make my own determination about which is best.

I looked around on Digikey for components that seemed to be able to handle AC, need a low activation voltage, and could handle a high load voltage.  The parts I came up with where these:

1. [Mechanical relay][7]
2. [Solid State Relay][8]
3. [Opto-isolator][9]

I placed the order and then ran the list by my friend who is an electrical engineer. (Flipping the order of those two would have made more sense, but I was anxious to get some parts.)  He pointed out that the main issue I would need to worry about was how much power dissipation each of these components could handle.  In my doorbell system, this could potentially be a lot.  I'll dig more into that in my next post.


[1]:{% post_url 2016-11-16-doorbell %}
[2]:http://www.explainthatstuff.com/howrelayswork.html
[3]:http://www.ni.com/white-paper/2774/en/
[4]:https://github.com/imightbeamy/buzzerbot9000/blob/master/how_to.md
[5]:https://www.renesas.com/en-sg/products/optoelectronics/technology/difference.html
[6]:http://lunainc.com/wp-content/uploads/2016/06/NSL-32SR2.pdf
[7]:http://www.digikey.com/product-detail/en/luna-optoelectronics/NSL-32SR2/NSL-32SR2-ND/5039808
[8]:http://www.digikey.com/product-search/en?mpart=PVA2352NPBF&v=448
[9]:http://www.digikey.com/product-detail/en/te-connectivity-potter-brumfield-relays/ORWH-SH-105D1F,000/PB2032-ND/4925028
[10]:https://twitter.com/paulcarletonjr/status/806360740273778688
