---
layout: post
title: "Morse Code to Emoji: A Brief history of character encodings"
---

# Introduction

I found myself wondering what Unicode symbols looked like at the bit level.  In the ASCII world, a single byte is a single character, so the encoding is relatively straightforward.  However, in the UTF-8 encoding, a single character can span multiple bytes, so how does the program know when the character starts and ends?

This question led me on a winding trail of history via Wikipedia and other sources through UTF-8, UTF-16 back to ASCII.  I thought I knew ASCII well, but found myself wondering, why is it laid out like this?  It turns out it is the way it is largely due to Teletype systems which are the way they are because of telegraphic typewriters which are finally based on the telegraph and therefore Morse code.  This seemed like a reasonable place to bottom out.  In this post, I'll walk through the different pieces of the story that led us all to :)

# 1825 - Samuel Morse

Our story starts in 1825 with Samuel Morse.  Morse was 33 years old and married to a woman named Lucretia.  Together, they had 3 children, the youngest of whom had just recently been born.  Morse was a renowned painter and was commisioned to paint a portrait of the revolutionary war hero Marquis de Lafayette (age 68 at the time).  Morse went to D.C. to paint the portrait.

In the midst of his painting, he received a letter from his father saying that his wife had been sick, but was recovering.  The next day, he received another letter stating that she had passed suddenly.  By the time he got back to New York, she had already been buried.  Morse was heartbroken and frustrated that his wife was suffering for days without his knowledge.  This set his mind toward improving the speed of communication.

# 1836 - Morse Code

Fast forward 10 years to 1836, and Morse has made friends with Joseph Henry and Alfred Vail.  Together, they develop the first electrical telegraph system.  Morse originally wanted the system to transmit only numerals.  The operators would then look up the numerals in a code book to see which characters the corresponded to.  Alfred Vail convinced him to include letters.  In order to determine which letters were most common and therefore should have the shortest encoding, Vail went to his local newspaper and counted the letters in their movable type system.

# 1870 - Baudot Code

The next character encoding came from Emile Baudot.  Baudot realized that the typical telegraph line was idle most of the time when transmitting a message.  He wanted a way to increase the efficiency, so he developed a system of time multiplexing.  Each of 5 different messages could be sent simultaneously across the same wire by splitting up their characters and interleaving their transmission.  This interleaving or multiplexing (and subsequent de-multiplexing) was achieved through a system of synchronized clockwork devices.

In order for his system to work, each character needed to be transmittable in the same amount of time.  To achieve this, he developed a 5 bit encoding with a corresponding 5 bit keyboard.  The keyboard had 3 keys for the right hand and 2 keys for the left.  Baudot designed his code in the interests of making it easy for the operator to remember and easy to input into the machine.

# 1901 - Murray-Baudot Code

In 1901, typewriters were becoming more popular and with it, typing training was becoming more common.  Donald Murray made the observation that if a typewriter could be made to send telegraphs, then all these people trained in typing could also be telegraph operators.  He designed a keyboard that mimicked the typical typewriter interface.  

In Murray's system, the operator would press a key corresponding to the letter they wanted to transmit, and the machine would handle translating it into the appropriate bits.  Since the operator no longer needed to enter the code, Murray made some modifications to Baudot's code to again make the most common letters contain the fewest "on" bits.  This was because each "on" bit required a mechanical operation in the machine, and fewer mechanical operations meant the machine would last longer.  Murray's modifications also added "control" characters like "carriage return" and "line feed" which we still see in encodings today.

# 1924 - ITA2

In 1924, the Murray-Baudot code was standardized by the ITU (International Telegraph Union) into ITA-2 (there was an ITA-1 before it, but ITA-2 stuck around for longer).

# 1963 - ASCII

ASCII was a 7 bit code which increased the number of bits from 5.  
