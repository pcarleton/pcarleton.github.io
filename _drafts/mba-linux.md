---
layout: post
title:  "Installing Linux on a MacBook Air"
---

# Background
After successfully repairing my 2011 MacBook Air (see my [last post][lastpost]), I had a decision to make: update to the latest OS X meant for beefier hardware, or install a lighter weight Linux operating system.  Having seen a co-worker's older MacBook Air bite the dust on the latest OS X version, and lacking any Linux in my life, I decided to start down the Linux trail.

# Picking A Distribution

I settled on [Debian](https://www.debian.org/) as my distribution. I ran Fedora and Ubuntu on my machines in the past and liked them well enough.  I briefly considered Arch Linux because it seemed pretty hardcore (there was a whole [streaming event](http://twitchintheshell.com/)) around installing it after all), but a friend of mine had recently settled on Debian after a frustrating wrestling match with Arch Linux, so I decided to join him.

# Making the Install Stick
I found a [set of instructions][install] on [Macworld.co.uk](https://Macworld.co.uk) which showed how to turn an .iso file into a bootable USB stick.  The instructions were for Ubuntu, but I was able make it work for Debian (I will list the exact commands I used below.)

I downloaded the Debian amd64 DVD `.iso` files from [here][deb-torrent]  The "amd" thing threw me for a loop since the processor in this machine is made by intel, but never fear, "amd64" simply indicates a 64 bit operating system ([more explanation][32v64]).  I didn't have a torrent client, so I downloaded [uTorrent](https://utorrent.com) which worked well.

I turned the multiple `.iso` files into a single `.iso` file with the following command:


```
cat debian-8.5.0-amd64-DVD-1.iso debian-8.5.0-amd64-DVD-2.iso debian-8.5.0-amd64-DVD-2.iso >debian.iso  
```

I next turned the `.iso` file into a `.dmg` file using a command from the [instructions][install] I mentioned above (the resulting file is `debian.img.dmg`):

```
hdiutil convert -format UDRW -o debian.img debian.iso
```


I then used the `dd` command from the [instructions][install] I mentioned above to copy the file my USB stick.  Note: It's important you get the drive number correct or you may overwrite something unintentionally.  Use `diskutil list`, and try unplugging your stick and plugging it back in to make sure you have the right one.

```
sudo diskutil unmountDisk /dev/disk2
sudo dd bs=1m if=debian.img.dmg of=/dev/disk2
```

I got the expected Alert saying the disk was not readable.  I rebooted my Mac while holding the option key, clicked the EFI boot option, and started installing Debian.

# Installation
During the Debian installation, I was prompted to insert a CD with the drivers for my Broadcom wireless card.  I did not have the drivers, and I only had the one USB stick handy, so I skipped this step with the plan of starting out sans-WiFi until I could transfer the drivers over.

The issue was that Debian is committed to only including Free and Open Source Software (FOSS) components in its core distribution.  Broadcom does not publish open source drivers, so drivers are not available in the install image.  However, Debian does allow access to these "non-free" drivers through its "non-free" package repositories.

I also ignored the prompts to enter my network settings and to update my packages (since their was no internet connection with which to do that.)

My computer booted to Debian, and I set about trying to install the wireless drivers.  The first order of business was to make my user account able to do the `sudo` command.  I followed the [sudo][sudo] page on the Debian site which gave me this:

```
su
# Enter root password
adduser paul sudo
```

I found the model number of the wireless chip to be a Broadcom BCM4322 via an [iFixit teardown][teardown].  I discovered that [wl][wl-module] was the preferred method of installing the drivers after reading through [a forum discussion][forum].  I noted that `wl` recommended having `wireless-tools` installed, which I did not (I checked this with `sudo apt list | grep wireless-tools`).  However, apt knew about the package which meant it must have been included in my install DVD.

I used `sudo apt edit-sources` to examine where `apt` was looking for packages.  It assumed I had a CD drive mounted with the install disk based on this line:

```
deb cdrom:[Debian GNU/Linux 8.5.0 _Jessie_ - Official amd64 DVD Binary-1 20160604-15:35]/ jessie contrib main
```

I redirected this to the path of my USB stick by changing that line to:

```
deb file:/media/paul/"Debian 8.5.0 amd64 1" jessie contrib main
```

I then installed wireless tools with `sudo apt-get update && sudo apt-get install wireless-tools`.


I plugged my usb stick back into my internet connected computer.  I reformatted it and put the .deb file for the [broadcom-sta-dkms][broadcom-drivers] package on it (that package contained the `wl` module).  I ran some `dpkg` commands to install the `.deb` file manually from the usb stick and then enable the drivers:

```
sudo dpkg -i /media/paul/usb/broadcom-sta-dkms.deb
sudo apt-get install -f
sudo modprobe -r b44 b43 b43legacy ssb brcmsmac bcma
sudo modprobe wl
```
(TODO: Figure out what's in a `.deb` file)

I was then able to use the GNOME network manager to connect to my network successfully.  Next, I used `sudo apt edit-sources` to point apt towards the repositories listed on [the wiki][sources-list].

The next thing I did was follow adjust my trackpad settings based on the tips listed [here][trackpad].  The palm detection was huge as well as the tap to click.

And now I have a working MacBook Air running Debian!

[lastpost]:({% post_url 2016-08-21-mba-repair %})
[deb-torrent]: http://cdimage.debian.org/debian-cd/8.5.0/amd64/bt-dvd/
[32v64]: http://askubuntu.com/questions/54296/difference-between-the-i386-download-and-the-amd64
[broadcom-drivers]:https://packages.debian.org/jessie/broadcom-sta-dkms

[sources-list]:https://wiki.debian.org/SourcesList
[trackpad]:http://uselessuseofcat.com/?p=74
[forum]:http://forums.debian.net/viewtopic.php?f=7&t=115650
[sudo]:https://wiki.debian.org/sudo
[install]:http://www.macworld.co.uk/how-to/mac/how-install-linux-on-mac-3637265/
[teardown]:https://www.ifixit.com/Teardown/MacBook+Air+13-Inch+Mid+2011+Teardown/6130#s26668
[wl-module]:https://wiki.debian.org/wl#supported
