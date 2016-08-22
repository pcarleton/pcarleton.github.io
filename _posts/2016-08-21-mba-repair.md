---
layout: post
title:  "Fixing a MacBook Air After a Spill"
---

# TL;DR

I spilled on my laptop, and decided to fix it. I found a [step-by-step guide][guide], the [tools required][toolkit], and the [replacement part][pbm-part] (and later a [cheaper option][pbm-keyboard]). The replacement was a success, cost $280 and took about 2 hours.

# Background

About 2 years ago, I spilled beer all over my laptop (a mid 2011 MacBook Air 13").

Soon after, the keyboard starting acting funny.   Pressing some of the keys felt a crunchy, pushing one key would register as pushing several, and other keys would not register at all.

I put the laptop on life support: I stuck it in a drawer and plugged in an external keyboard and monitor, and I got a [KVM switch][kvm-switch] so I could switch between my laptop and my desktop.

Fast forward 2 years, my KVM setup is no more, and my laptop has been gathering dust.  I decided I needed to either get this thing working or get rid of it.  I had 3 options: sell it, replace the keyboard, or fix the keyboard.

# Option #1: Sell For Parts
I briefly considered selling the MacBook Air for parts. On [ebay][ebay], the going rate for a broken MacBook Air was about $200.  I also asked for a quote at [macbookmoney.com](https://macbookmoney.com), and they offered my $120 with free shipping.

This was a decent chunk of change, but I wanted a personal laptop, and $200 wasn't going to get me anything very good.  I wanted to know how much it would cost to repair.

# Option #2: Replace the Keyboard

#### Teardown Guide
I found a guide for replacing the top case of a MacBook Air on [iFixit.com][guide].  The steps looked relatively straightforward with the right set of tools.  The pictures got me pretty excited about tearing my laptop down.

#### Tools
I had heard of [iFixit](http://ifixit.com) before and thought the toolkits they sold were cool.  Whenever I broke my phone's screen, or my laptop's hard-drive crashed, I felt tempted to buy a toolkit and the part, but the cost always came out to be too much.  This time around, I decided that the enjoyment I got out of taking apart my laptop would be worth the money I put towards the kit.  As an added benefit, any future repairs will have a lower upfront cost since I'll already have the tools.  I went with the [Pro Tech Toolkit][toolkit] for $80. The [teardown guide][guide] lists the individual tools needed too, if you don't want to shell out for the full kit.

#### Parts

iFixit sells the [top case][ifixit-part] for $300 new or $200 used, but the part was out of stock when I looked.  I poked around and found the part on [PowerBookMedic.com](https://PowerBookMedic.com) for $200.  I have since learned that you can buy just the [keyboard assembly][pbm-keyboard] for about $50 which is what I would try first if I did this same repair again.  

When buying from PowerBookMedic, I recommend using their [serial number compatibility checker][serial-no].  I did not do this initially, and I ended up ordering a MacBook Air 11" instead of a 13", d'oh!  I actually did not realize my mistake until I had completely disassembled my computer and was ready to pop in the replacement.  So I also recommend you check your part before starting on your repair!

I talked to PowerBookMedic support, and they were nice enough to let me return the wrong case and get a new one.  I just had to pay shipping and the price difference.  This should have been a warning sign to me, because there actually was no price difference in the 13" and 11" top cases.  Several weeks later, I got my new part, and was excited to put it all together when I found out that they had mistakenly sent me a 13" MacBook Pro top case.  Luckily, I checked this time before disassembling.  They again were very accommodating and rush delivered the correct part to me and refunded me the price difference they had charged me.  Their support was really great.

# Option #3 Fix the Keyboard
 A friend who is an electrical engineer suggested that I may not need a replacement at all.  Since the keyboard is mostly mechanical, and everything else in my computer was working fine, he suggested that the problem was likely that some dried up beer was affecting the metal contacts in the keyboard assembly.

 The first time tearing it down, I had the wrong replacement part, so I figured I would give cleaning it a shot.  I got the highest % isopropyl alcohol that Walgreens had.  I let the keyboard sit in a tub of it, and then tried brushing away some of the gunk with q-tips. It was now much cleaner that it was before.

I re-assembled the computer, but the keyboard still did not work.  However, it did not work in exactly the same way it was broken before.  This gave me confidence in my ability to take apart the laptop and put it back together without totally messing it up.  It's possible a more thorough cleaning with some better scrubbing would do the trick, so I recommend trying that.

# Parting Thoughts
Laptops are surprisingly sturdy.  There were a few  moments during my first tear down when I thought I had broken everything.  At one point, I stripped a screw on the touch pad so bad that I ended up using a power drill and drilling the screw's head off.  This actually worked surprisingly well. While you do need to be careful, know that these things were built to be repaired by someone, so that someone may as well be you.

Make sure to flip  up the retaining flaps on your ZIF sockets before taking the cables out and to flip them down after putting the cable back in.  I did not do this on my first tear down. Taking the tabs out was easy without flipping up the flap, so I had no idea I was doing it wrong.  Then when putting it back together, I was real frustrated because I could not get the ribbons tab on the track-pad to go back in.  After looking really close at a repair video, I realized what I was doing wrong.  I flipped the tab up, and it was no problem to get the ribbon back in.

All in all the repair went well, and I will definitely be repairing the next piece of electronics I break.  I highly recommend giving it a shot next time you have a mishap like mine.

Checkout my next post where I explain how I installed Linux on my newly repaired laptop.

[guide]: https://www.ifixit.com/Guide/MacBook+Air+13-Inch+Mid+2011+Upper+Case+Replacement/9427
[toolkit]: https://www.ifixit.com/Store/Tools/Pro-Tech-Toolkit/IF145-307-1
[ifixit-part]: https://www.ifixit.com/Store/Mac/MacBook-Air-13-Inch-Mid-2011-Upper-Case-with-Keyboard/IF188-062-3
[pbm-part]: http://www.powerbookmedic.com/MacBook-Air-13-Top-Case-Keyboard-Assembly-p-20002.html
[pbm-keyboard]:http://www.powerbookmedic.com/MacBook-Air-13-Keyboard-with-Backlight-Module-p-38989.html#buy_together
[kvm-switch]: https://www.amazon.com/gp/product/B00006B8EC/ref=oh_aui_search_detailpage?ie=UTF8&psc=1
[serial-no]: http://www.powerbookmedic.com/identify-mac-serial.php
[ebay]:http://www.ebay.com/sch/i.html?_from=R40&_trksid=p2050601.m570.l1313.TR12.TRC2.A0.H0.Xbroken+macbook+air.TRS0&_nkw=broken+macbook+air&_sacat=0
