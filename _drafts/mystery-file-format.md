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

# The Mystery Photo Format

I recently received an email from my grandpa.  He found a DVD with some old pictures of my brother's graduation, but he could not open some of the files.  I asked if he would send me a few of the files so I could take a crack at figuring out what was wrong with them.

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

 To investigate if any of these magic bytes occurred in my file, I used the tool `hexdump` to translate the contents of my file into hexidecimal. The "\\377\\330\\377" translates to FF D8 FF in hexidecimal.

```
$ hexdump IMGP0175.JPG | head -n 3
0000000 0000 ba01 e245 f41e 01f4 8901 f8c3 0000
0000010 e001 ec07 0080 bd00 25e9 0376 83b8 686c
0000020 506e e578 3405 88a1 0f58 7ff8 c4d5 0d30
```

Unfortunately, this hex pattern was no where to be found in my file.

I then considered that the file might be compressed, so I wondered if there was a way to detect that.  I searched around, and sure enough, the `file` command line utility does just that.

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
``


[1]:http://www.imagemagick.org/discourse-server/viewtopic.php?f=1&t=15566
[2]:{% post_url 2016-08-28-whats-in-a-deb %}
