---
layout: post
title: "Debugging a Doorbell"
---

- Hack my Doorbell
- get a notification when someone buzzes
- buzz someone in from my phone (or alexa/google home)

- Starting point: 3 wires coming in, some coils, a hammer, and a button

- Measured 19V AC over the terminals of the button

- Uh oh... I don't really get AC
- Still don't really...
- What does completing the circuit mean?
- I'll write what I know so far, and fill in the blanks.

- Started watching some videos on Khan academy about circuits

- Found the digikey circuit builder

- Found some articles about how people have done this, the used an optocoupler
- Depends a lot on your set up

- Door bell transformers commonly go from 110 VAC to 20 VAC


# The Problem

I live in an apartment with a front lobby and a buzzer that lets people in.  Having a button to push to remotely unlock a door is pretty neat, but you know what's neater? Being able to do it from your phone.  That's what I set out to do in this project.

I saw a post on [Twilio's blog][1] with a run down of quick rundown of how they did their system. (Twilio is a company that does messaging API's for phones, so you can do things like have a server respond to SMS messages).  The article suggested using an optocoupler, something I had never heard of.  I found a good explanation of one [here][2] and the specs for one [here][3].  The gist of it is that an optocoupler provides isolation between the doorbell circuit and the circuit that your Raspberry Pi or Arduino is running on.  When current runs through one end, a light turns on which the other end detects and allows current through its end.  So when someone hits the doorbell, a light inside the optocoupler goes on, the recieving side sees it, and that signal gets propagated to your RPi all without subjecting your board to the high powered stuff coming out of your wall.

That was well and good, but I needed to know where to put this thing in my system. I also needed to know what kind of voltages I was dealing with.  Luckily, my friend Jason is an electronics wizard (see electrical engineer).  He let me borrow his Multimeter, and I set to work.

Here is a picture of my doorbell:
<insert image here>

Here it is in terms of the individual components.  I made this with the [Digikey circuit builder][4]. We have 2 buttons: the outside door bell, and the inside button for the door buzzer.  We have 2 coils which affect the hammer that rings the bell. These are inductors.  We have the mechanism that unlocks the door which I'm calling a resistor.  And finally we have the big giant metal piece which I'm calling the platform (and using the European resistor symbol for.)
<components>

Here are the components wired up.  There are three wires coming in, one goes to the button, one goes to the "platform" and the third goes to a screw in the middle of the platform but isolated from the platform.
<wired up>


The first thing I measured was across the button.  I measured 19 V AC.  Uh oh.  I remembered mostly how DC works from taking physics, but whatever I learned about AC was out the window.  Here's what the diagram looks like adding in the connections we know about wired up:


Based on this diagram, I was confused.  When I pushed the inside button, why didn't the bell ring?


I enlisted Jason's help.  We discovered another component which was a switch between the inductors and the hammer.  When we put a piece of paper in between the switch, the bell didn't ring.  This led us to the following diagram:
<doorbell switch>


After thinking about how it could fit together we came up with this final diagram:
<doorbell>


With the system diagrammed, it was time to decide how to hook it up.


I started watching some Khan academy videos and will report back when I understand more.

I pressed on and enlisted the help of Jason to figure out how it all fit together.  Here's where the components look like with
<raspberry pi>


- full wave rectifier
- relay
