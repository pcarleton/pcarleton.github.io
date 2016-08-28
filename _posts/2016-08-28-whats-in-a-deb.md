---
layout: post
title:  "What's In A .deb"
---

I recently installed Debian on my MacBookAir (see my [last post][mba-linux-post] for details).  As a part of that exercise, I touched some `.deb` files which got me wondering, what's in a `.deb` file?  

# The `.deb` File Format
[Wikipedia](https://en.wikipedia.org/wiki/Deb_(file_format)) tells us:
>Debian packages are standard Unix ar archives that include two tar archives optionally compressed with gzip (zlib), Bzip2, lzma, or xz (lzma2): one archive holds the control information and another contains the program data.

Later it mentions a third member of the archive called `debian-binary` which indicates the deb format version.  This is "2.0" for all current versions of Debian.


# The `ar(chiver)` Tool

[Wikipedia](https://en.wikipedia.org/wiki/Ar_(Unix)) says this about `ar`:
>The archiver, also known simply as ar, is a Unix utility that maintains groups of files as a single archive file. Today, ar is generally used only to create and update static library files that the link editor or linker uses and for generating .deb packages for the Debian family; it can be used to create archives for any purpose, but has been largely replaced by tar for purposes other than static libraries.

We can see it's uses have mostly been replaced by the `tar` tool (which turns out is short for `(t)ape (ar)chive`).  Aside from Debian packages, the `ar` tool is sometimes also used by gcc to group many `.o` object files into a single `.a` file.

Note that the description makes no mention of compression.  We can test out the format by writing our own archive file with some text files to see what comes out.  The `ar` commands we want are `Insert files into archive with (r)eplacement`, `(c)reate the archive` and `Add an index to the archive ... (s)`.

```
# Create dummy files
% echo "123" >1.txt
% echo "456" >2.txt
# Make the archive
% ar rcs test.a 1.txt 2.txt
# Inspect the contents
% cat test.a
!<arch>
1.txt/          1471876258  1000  1000  100644  4         `
123
2.txt/          1471876266  1000  1000  100644  4         `
456
```


First there's the 'global header' of "!<arch>".  Then for each of the input files, there's some metadata, then the contents of the file.  The first piece of metadata is the file's name.  Next there is the modification timestamp:

```
% date --date='@1471876258'
Mon Aug 22 07:30:58 PDT 2016
```

Then there is the owner ID and user ID:

```% echo $UID
1000
```

Then there's the file mode in octal. 100644 in this case means `-rw-r--r--`.  And finally there is the file's size in bytes.

So the `ar` tool groups files together into a single file without compression and preserving their order and file names.

It seems like it would make sense to switch to `tar` or some other tool which offers compression, but I imagine they have some motivation to stick with the `ar` tool.

# Taking apart the Atom `.deb` File

I decided to check this out on the latest `.deb` file I had downloaded: [Atom][atom], the text editor.

I extracted the archive members using `ar`, and here's what I saw:

```
% ar x atom-amd64.deb
% ls
atom-amd64.deb  control.tar.gz  data.tar.gz  debian-binary
% cat debian-binary
2.0
% tar -xvf control.tar.gz
./
./control
% cat control
Package: atom
Version: 1.9.9
Depends: git, gconf2, gconf-service, libgtk2.0-0, libudev0 | libudev1, libgcrypt11 | libgcrypt20, libnotify4, libxtst6, libnss3, python, gvfs-bin, xdg-utils, libcap2
Recommends: lsb-release
Suggests: libgnome-keyring0, gir1.2-gnomekeyring-1.0
Section: devel
Priority: optional
Architecture: amd64
Installed-Size: 224476
Maintainer: GitHub <atom@github.com>
Description: A hackable text editor for the 21st Century.
Atom is a free and open source text editor that is modern, approachable, and hackable to the core.
```
There's the deb format version number of "2.0".  Also there's the control and data archives as tarballs.  In the control archive we have a text file with metadata about the package including dependencies and a description.  (Interestingly there's no dependency on nodejs!).

In the data tarball, there's a whole bunch of stuff.  Here is the directory structure:

```
% tree -L 4 -d usr
usr
├── bin
└── share
   ├── applications
   ├── atom
   │   ├── chromedriver
   │   ├── locales
   │   └── resources
   │       ├── app
   │       └── app.asar.unpacked
   ├── doc
   │   └── atom
   ├── lintian
   │   └── overrides
   └── pixmaps

14 directories
```

This structure mimics where the files ended up on my machine after installing.  In a follow up post, I'll take a look at how to create our own `.deb` file and also the different compression modes for `tar`.

[mba-linux-post]:{% post_url 2016-08-25-mba-linux %}
