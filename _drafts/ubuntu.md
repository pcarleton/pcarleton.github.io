---
layout: post
title: "Switching to Ubuntu from Debian"
---

# From Debian to Ubuntu

I recently decided to switch from Debian to Ubuntu on my Macbook air (see [my previous post][1] about initially installing Debian).  The two situations that prompted this were 1) trying to work with ROS and finding that installing with Ubuntu would be easier and 2) working with OCaml and running into an issue with system packages.  Neither of these is a very compelling reason since I am sure there were ways to get both to work in Debian.  Some times you want to dig deep into why something is not working, and some times you want to just switch to something where it does work.

# Bootable USB stick w/ other partition

I first downloaded an install image from the [Ubuntu page][2].  I only had one USB stick, and I had some files on it that I wanted to keep.  I wasn't sure if this was possible since most instructions tell you to reformat the USB stick.  Fortunately [an answer][3] on AskUbuntu that suggested it was possible.  It recommended using [gparted][4] (Gnome PARTition EDitor), so I installed that with `sudo apt-get install gparted`.  I was able to shrink the partition that had the files I wanted to keep, and create a new partition to put the Ubuntu live image.  `gparted` also lets you set the "boot" flag on a partition (I'm not actually sure if this step was necessary based on what happened later).  When I was attempting to set the boot flag, the "Manage Flags" button was grayed out.  Thanks to a helpful [superuser][5] post, I learned that you have to apply your partition edits before you can mess with flags.

# Booting From the Grub Command line

I ran into problems booting from the USB which may have been related to the fact that my image was on a secondary partition.  When I tried restarting and getting to the boot [menu by pressing the OPTION key][6] on my mac, I wasn't seeing my USB stick pop up.

Since I already had GNU GRUB installed (GNU GRand Unified Bootloader) from my Debian install, I could get to the grub command line.  I got a quick crash course on GRUB from this [very thorough post][7] and [its follow-up][8].  From this I learned about 2 important pieces to the boot process.  The first is `initrd` which is a sort of generic bootstrapping filesystem which adapts the Linux kernel to whatever hardware is present.  The second is actual kernel image which is called `vmlinuz`.  I plan on doing a post which digs into how the `initrd` works and also how the filesystem behind the live USB image works, so stay tuned.

I got to the GRUB command line by hitting `c` at the GRUB prompt.  From here, my task was to find the USB drive, and then find the `initrd` and `vmlinuz` files.  Typing `ls` at the prompt will list the available partitions like `(hd0,gpt1) (hd0,gpt2)` etc.  The first item in the parentheses is the hard drive, and the second is the partition number.  It's not clear to me how these numbers get assigned, but you can inspect the contents of the drives by typing `ls (hd0,gpt1)` for instance.  If GRUB can read the drive, it will list its contents.  By poking around, I was able to find the USB drive.  It had label `(hd1,gpt2)` for the partition with my backup files and `(hd1,gpt1)` for the partition with the Ubuntu install disk.

I looked at a [couple][9] [posts][10] about how to boot from the USB command line.  Setting the root and then using `chainloader +1` did not work for me, I think because the `vmlinuz` and `initrd` files are not in the top level directory on the drive, they are under the "casper" directory. I located these by trial and error with "ls" in the top level directories.  Using the `linux` and `initrd` commands worked partially:

```
$ linux (hd1,gpt1)/casper/vmlinuz.efi
$ initrd (hd1,gpt1)/casper/initrd.lz
$ boot
```

It booted, but failed with a message saying no "init" was specified and left me at a "initramfs" prompt.  I did not know what to do here, so I dug around on the Ubuntu side and found [some grub configurations][11] for booting from an ISO image.  This wasn't exactly what I was doing since it wasn't an ISO image, but a USB stick.  However, the boot config looked similar to what I had been trying:

```
menuentry 'ISO Precise ' {

set isofile="/iso/ubuntu-12.04-desktop-amd64.iso"

loopback loop (hd0,5)$isofile

linux (loop)/casper/vmlinuz boot=casper iso-scan/filename=$isofile noprompt noeject

initrd (loop)/casper/initrd.lz

}
```

Ignoring the "iso-scan" argument, the interesting piece I saw was the "boot=casper" argument.  From the [man page][12], `casper` is a "hook" for initramfs-tools to create a filesystem suitable for booting for a live image.  Under the `/casper` directory, there is a filesystem in `squashfs` format (a special compressed read-only format) and create a writeable filesystem using `unionfs`.  My knowledge of `squashfs` and `unionfs` is limited (I do remember reading that Docker leverages `unionfs`), so I will dig into that more another time.  For now, setting the `boot=casper` argument for the `linux` command did the trick:

```
$ linux (hd1,gpt1)/casper/vmlinuz.efi boot=casper
$ initrd (hd1,gpt1)/casper/initrd.lz
$ boot
```

This booted successfully, and then installation was a breeze. Notably, I did not have to do some USB gymnastics to get a driver for my wireless card since Ubuntu does not have the same ideological barriers against closed source drivers.  It looks like the modules Ubuntu uses are the `brcmsmac` drivers:

```
$ lsmod | grep -e b43 -e b44 -e brcmsmac -e bcma -e ssb -e wl
brcmsmac              573440  0
cordic                 16384  1 brcmsmac
brcmutil               16384  1 brcmsmac
b43                   417792  0
mac80211              737280  2 b43,brcmsmac
cfg80211              565248  3 b43,brcmsmac,mac80211
ssb                    65536  1 b43
bcma                   53248  2 b43,brcmsmac
```

Some other perks of switch were that my brightness and volume buttons work as expected, and my trackpad didn't need any adjusting.  I uploaded the output of `synclient` on Ubuntu and Debian to a [gist][13].  When I compared the two, the most glaring difference was that Ubuntu sets `TouchpadOff=2`.  From the [man page][14], this setting means:

> Option "TouchpadOff" "integer"
    Switch off the touchpad. Valid values are:
    0	Touchpad is enabled
    1	Touchpad is switched off
    2	Only tapping and scrolling is switched off
    Property: "Synaptics Off"

Interestingly, I am still able to tap and scroll, but my palm does not cause nearly as many annoying issues as it did in Debian.

# Configuring From Scratch

Next I had to install all the new packages, set up my dotfiles, python virtual environments etc.  I decided to script all this as I went so I didn't have to remember all the things I need to install the next time I'm starting from a fresh OS install.  The scripts I used are pretty crude, but they're available on my github [here][15].

# Conclusion

Overall, I am happy with the change to Ubuntu and picked up a few lessons along the way along with some topics for future blog posts.


[1]:{% post_url 2016-08-25-mba-linux %}
[2]:https://www.ubuntu.com/download/desktop
[3]:http://askubuntu.com/questions/423300/live-usb-on-a-2-partition-usb-drive
[4]:http://gparted.org/
[5]:http://superuser.com/questions/833779/gparted-has-grayed-out-manage-flags
[6]:https://support.apple.com/en-us/HT204417
[7]:http://www.dedoimedo.com/computers/grub.html
[8]:http://www.dedoimedo.com/computers/grub-2.html
[9]:http://blog.viktorpetersson.com/post/93191892924/how-to-boot-from-usb-with-grub2
[10]:http://superuser.com/questions/936889/booting-a-usb-from-the-grub-command-prompt
[11]:https://help.ubuntu.com/community/Grub2/ISOBoot/Examples
[12]:http://manpages.ubuntu.com/manpages/xenial/man7/casper.7.html
[13]:https://gist.github.com/pcarleton/3e0bace4cf8df20e8d169b816af93192
[14]:https://www.x.org/archive/X11R7.5/doc/man/man4/synaptics.4.html
[15]:https://github.com/pcarleton/initialinstall
