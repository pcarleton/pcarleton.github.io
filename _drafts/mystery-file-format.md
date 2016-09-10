---
layout: post
title:  "Solving a File Format Mystery"
---

## TL;DR

The `file` command line utility is really useful if you're not sure what format a file is. For example:

```
$ file IMGP0175.JPG
IMGP0175.JPG: MPEG sequence, v2, program multiplex
```

# A Mystery Photo Format

I recently received an email from my grandpa.  He found an old DVD with pictures from my brother's graduation, but he could not open some of the files.  I asked if he would send a few of them my way so I could take a crack at figuring out what was wrong with them.

He sent me a file called `IMGP0175.JPG`.  I searched around and found that ImageMagick (`sudo apt-get install imagemagick`) had a tool called `identify`.  From the man page (`man identify`):

>identify - describes the format and characteristics of one or more image files.

This seemed promising, so I ran it on my file:

```
$ identify IMGP0175.JPG
identify: Not a JPEG file: starts with 0x00 0x00 `IMGP0175.JPG' @ error/jpeg.c/JPEGErrorHandler/322.
```

This told me what I already knew, it wasn't a good JPEG.  I was hoping it would try to guess at what other format it was (spoiler: there's a tool that will).  I continued searching and found [a forum post][1] by someone in a similar predicament.  That person's problem turned out to be that the file had some preceding garbage bytes.  A helpful poster indicated that the leading bytes that ImageMagick uses to determine file format are available in a config file.  It was in `/etc/ImageMagick-6/magic.xml` on my machine.  The poster indicated that he found that the JPEG leading bytes showed up after 21 bytes of garbage, and once they were removed, the image would open.

Here is the relevant portion of the `magic.xml` file:

```
<magicmap>
  <!-- <magic name="GIF" offset="0" target="GIF8"/> -->
  <!-- <magic name="JPEG" offset="0" target="\377\330\377"/> -->
  <!-- <magic name="PNG" offset="0" target="\211PNG\r\n\032\n"/> -->
  <!-- <magic name="TIFF" offset="0" target="\115\115\000\052"/> -->
</magicmap>
```

 To investigate if any of these magic bytes occurred in my file, I used the tool `hexdump` to translate the contents of my file into hexidecimal. The "\\377\\330\\377" translates to `FF D8 FF` in hexidecimal.

```
$ hexdump -C IMGP0175.JPG | head -n 3
00000000  00 00 01 ba 45 e2 1e f4  f4 01 01 89 c3 f8 00 00  |....E...........|
00000010  01 e0 07 ec 80 00 00 bd  e9 25 76 03 b8 83 6c 68  |.........%v...lh|
00000020  6e 50 78 e5 05 34 a1 88  58 0f f8 7f d5 c4 30 0d  |nPx..4..X.....0.|
```

(Note: I used the -C option to get one-byte display.  Without this option, `hexdump` will return 2-bytes at a time and display them as little-endian.  Practically, this means the beginning of the file below would be displayed as `0000 ba01`. Thanks to [this stackexchange post][4] for clarifying that.)


Unfortunately, the `FF D8 FF` hex pattern was no where to be found in my file.

I then considered that the file might be compressed, so I wondered if there was a way to detect that.  I searched around, and sure enough, the `file` command line utility does just that.  From `file` man page:

>   file — determine file type

The result for my file:

```
$ file IMGP0175.JPG
IMGP0175.JPG: MPEG sequence, v2, program multiplex
```

It turns out it was a video file this whole time.  I renamed it to have a ".mpg" extension, and successfully opened it in VLC. Mystery solved!


#### Other Formats

I was curious what `file` would tell me about the `.deb` files I had been digging into in a [previous post][2]

```
$ file atom-amd64.deb
atom-amd64.deb: Debian binary package (format 2.0)
```

I was expecting it to tell me it was an `ar` file, but it went even further.  The output on an `ar` archive is:

```
$ file test.ar
test.ar: current ar archive
```

#### What is this strange magic?

I was curious how the `file` utility does its magic.  The `man` page describes a 3 different tests it does.  It first examines the output of `stat`, it then looks for some magic bytes in the beginning of the file, and finally checks if it matches a known text character encoding.

The `man` page explained that `stat` would indicate a symbolic link, but also a "special" file.  I was curious what this would be like, so I started looking for a file that would be considered "special".  I discovered that the file descriptors for running processes exist in the `/proc/<pid>/fd` directory.  (You can get the process id, or pid, by running `ps aux | grep <process name>`).  I started running `stat` on the file descriptors of random processes on my machine, but I kept finding symbolic links to broken pipes.  I then decided to run it on the descriptors of my currently running shell process, and voila:

```
$ stat /proc/12156/fd/0
  File: ‘/proc/12156/fd/0’ -> ‘/dev/pts/0’
  Size: 64              Blocks: 0          IO Block: 1024   symbolic link
...

$ stat /dev/pts/0      
  File: ‘/dev/pts/0’
  Size: 0               Blocks: 0          IO Block: 1024   character special file
...

$ file /dev/pts/0
/dev/pts/0: character special (136/0)
```

This showed how the `stat` test would be helpful.  I was next interested in the "magic" test since that is what would have triggered for my MPEG file since it is not a special file and it is not a text file.  The `man` page listed the locations to look for the magic file as `/etc/magic` and `/usr/share/misc/magic/magic.mgc`.  The first location was a file with just a comment saying I could put local magic data in there in a format described in magic.  The `man` page for `magic` describes the format to use.

As for the `magic.mgc` file:

```
$ file /usr/share/misc/magic.mgc
/usr/share/misc/magic.mgc: symbolic link to ../file/magic.mgc

$ file /usr/share/file/magic.mgc
/usr/share/file/magic.mgc: magic binary file for file(1) cmd (version 12) (little endian)
```

It's a binary file, so it is not clear what specific pattern is being applied in my situation.  Luckily the `file` source code is available on github.  Here are [the lines][3] that correspond to the "MPEG v2 sequence":

```
# MPEG sequences
# Scans for all common MPEG header start codes
...
0        belong&0xFFFFFF00  0x00000100
>3       byte               0xBA           MPEG sequence
!:mime  video/mpeg
>>4      byte               &0x40          \b, v2, program multiplex
>>4      byte               ^0x40          \b, v1, system multiplex
```

The format of the file is in 3 columns, a byte offset, a format, and a test.  Additionally, the `>` characters represent a hierarchy as explained in the `magic` man pages:

>The number of > on the line indicates the level of the test; a line with no > at the beginning is considered to be at level 0.  Tests are arranged in a tree-like hierarchy: if the test on a line at level n succeeds, all following tests at level n+1 are performed, and the messages printed if the tests succeed, until a line with level n (or less) appears.

Looking back at the MPEG lines, there are 3 tests that had to pass in order to observe the output that I saw.

__First Test__

```
0        belong&0xFFFFFF00  0x00000100
```

The first line says start at byte 0, take a big-endian long (4 bytes), take the bit-wise AND of those 4 bytes with 0xFFFFFF00, and check that it is equal to 0x00000100.  Taking a big-endian long basically means don't flip them around.  If we took a little-endian long, the 4th byte would be considered the first.  To illustrate this, I wrote a `lelong` line to the `/etc/magic` file and then created some test files:

```
$ cat /etc/magic
0       lelong  0x00000001        Paul Little-Endian Long

# Create test files
$ echo "01000000" | xxd -r -p >test0.dat
$ echo "00010000" | xxd -r -p >test1.dat
$ echo "00000100" | xxd -r -p >test2.dat
$ echo "00000001" | xxd -r -p >test3.dat

# Run file on the test files
$ file test*.dat                        
test0.dat: Paul Little-Endian Long
test1.dat: raw G3 data, byte-padded
test2.dat: data
test3.dat: data
```

I used the `xxd` tool with the `-r` option which turns an ascii hex dump into bytes.  The results show that a file with 0x01 as the first byte matched the test for a little-endian long matching 0x00000001

Back to the MPEG tests, the first 4 bytes of my file are `0x000001ba`. This AND'd with `0xFFFFFF00` is `0x00000100`, so we pass that check.

__Second Test__

```
>3       byte               0xBA           MPEG sequence
```

This line indicates that we should look at the byte at index 3 (the 4th byte) and check that it is equal to `0xBA`.  This passes.

__Third Test__
`>>4    byte    &0x40   \b, v2, program multiplex`  indicates that we should look at the byte at index 4 (the 5th byte) and ensure that it has its 2nd bit flipped.  The 5th byte of my file was `0x45`, so this check passed as well.  This means my file is an MPEG Sequence v2.


We can make a dummy file that passed all these checks by echoing 5 bytes, like this:

```
$ echo "000001ba40" | xxd -r -p >dummy.dat && file dummy.dat
dummy.dat: MPEG sequence, v2, program multiplex
```

And with that, we now know how the magic in the `file` utility works.  It's not magic at all, but the product of a lot of hard work by the maintainers to find the "magic" numbers for thousands of different file types.

[1]:http://www.imagemagick.org/discourse-server/viewtopic.php?f=1&t=15566
[2]:{% post_url 2016-08-28-whats-in-a-deb %}
[3]:https://github.com/file/file/blob/f27ea71acfe3bf9609c923d2980227484b7f1b20/magic/Magdir/animation#L195-L199
[4]:http://unix.stackexchange.com/questions/55770/does-hexdump-respect-the-endianness-of-its-system
