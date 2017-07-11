---
layout: post
title: "What does mount do?"
---

# Background

As I mentioned in my [post about containers][container-post], I want to write a container tool in rust to learn more about rust and containers.  I started doing that with my friend Kevin, and our (humble) progress so far is on Github in a project we called [bucket][bucket-repo] (like a container, that's sometimes rusty... or something).

We're following a long a talk given by Eric Chiang at CoreOS Fest called [Containers From Scratch][core-os-talk-video] ([slides][core-os-talk-slides]).  In the talk, Eric walks through the linux command line utilities that you can string together yourself to make a "container".  We got as far as creating a "root filesystem", `chroot`ing into it, and then making a separate PID namespace by calling `unshare`.  It was during this `unshare` step that I ran into an error message that led me to learn more about `mount` that I want to document here.

# What does `mount` do?

`mount` is a command line utility which wraps the `mount` system call. (Another fun fact I learned: the #'s after a name in a man page indicates what section that man page is in.  So [`mount(2)`][mount-2] is the system call because it is in the "System calls" section of the manual and [`mount(8)`][mount-8] is the command line utility because it is in the "System Administation Commands and Daemons" section. More info in this [SO Answer][man-page-numbers]).

If you're like me, you have had to call mount some times when you've plugged in a USB drive and it hasn't worked correctly.  Usually, the OS handles the mounting for you. For instance, when I plug in a USB stick to my computer, it pops up a Files window with the contents.

First, it's worth knowing how to get information about current mounts. The place for that is `mtab`.  `mtab` is a read-only file that lives at `/etc/mtab` (and in my case is a symlink to `/proc/self/mounts`) which lists the current active mounts.  When I plugged in my USB drive, I saw the following line at the bottom of `mtab`:

```
/dev/sdc2 /media/paul/P16G hfsplus ro,nosuid,nodev,relatime,umask=22,uid=0,gid=0,nls=utf8 0
```

If we break that down, we have a device on the far left called `/dev/sdc2`. (Quick side note: to break down `sdc2`, first ignore the `sd`, then you have `c` which indicates it's the third drive that the OS has seen.  `sda` is my computer's hard drive, I'm not sure what `sdb` is.  Then the `2` means what partition on that drive, so this is the 2nd partition. (more info in this [Superuser answer][so-device-names]))

After that, we have the mount point.  This is directory where the files on the device are accessible.  In this case, I can go to `/media/paul/P16G` to access the files on the USB stick.

Next is `hfsplus` indicating the filesystem type.  Then there are several options and then 2 0's indicating the dump/pass options.  0's mean we don't back up this mount (dump) and we don't run `fsck` on it to detect errors (pass).  The [`fstab` wikipedia page][wiki-fstab] has more info on this file's format.

If I want to unmount this, I can run `umount /media/paul/P16G`.  I can then remount it with `sudo mount -t hfsplus /dev/sdc2 /media/paul/P16G`. (Interestingly, I had to create the P16G directory since when I unmounted it, the directory went away.  I'm not sure why that is.)

Another interesting place related to `mount` is `fstab`.  It's the same format as `mtab`, but instead of reflecting the current mounts, it represents what mounts should be created at boot time.  If you change that file, you will have new mounts when you reboot.

That gives us enough information to move on to `unshare`.

# What does `unshare` do?

The particular `unshare` command we were looking at was:

```
sudo unshare -p -f --mount-proc=$PWD/rootfs/proc chroot rootfs /bin/bash
```

This creates a new PID namespace, mounts a `proc` filesystem under the `rootfs/proc` directory and then executes `chroot` and finally `bash`.

The tricky part I was running in to was that I was getting a "Invalid argument" error message when I tried to run this command.

First, I'll look at mounting a proc filesystem.  It turns out I can do this anywhere and I don't need a device like I did for mounting a USB drive.  Instead, whatever I pass in as the device becomes a kind of dummy label.  For instance, I can say `mount -t proc dummy ~/fs1` and I'll get a `proc` filesystem in the `fs1` directory.  If I do `ls ~/fs1`, it will look identical to doing `ls /proc`.  Additionally, I get the following line in `mtab`:

```
dummy /home/paul/fs1 proc rw,relatime 0 0
```

Now, I want to look at what the `unshare` command does when it has the `--mount-proc` command because it starts to demonstrate why I ran in to my error.  Here are some lines from the [unshare source code][unshare-source]:

```
if (procmnt &&
    (mount("none", procmnt, NULL, MS_PRIVATE|MS_REC, NULL) != 0 ||
     mount("proc", procmnt, "proc", MS_NOSUID|MS_NOEXEC|MS_NODEV, NULL) != 0))
          err(EXIT_FAILURE, _("mount %s failed"), procmnt);
```

I can see there are 2 calls to `mount(2)` (the system call).  The first isn't creating a mount, but it's taking the mount point and making it private recursively.  This is the same as saying `mount --make-private $procmnt`.  Interestingly, if I unmount `fs1`, and then try to make it private with `mount --make-private fs1`, it fails, saying that `fs1` is not a mountpoint.  This is not the exact error I was getting from `unshare`, but I assume it has the same source.  If I run `mount --bind fs1 fs1`, and then run the make private command it works (and similarly with the `unshare` command).  The problem was that the `unshare` command expects the target for mounting the new proc filesystem to already be a mount point.

# What does `--make-private` do?

It's nice to know why the command was failing and how to fix it, but what might be more important to understand is why `unshare` would be trying to make the `proc` filesystem private to begin with.

The [`mount(8)` manpage][mount-8] explains the different options for the sharing status of a file system so I won't repeat them here.  However, it's useful to know how to tell what the sharing bits are for a particular filesystem.  To get that, I can look at `/proc/self/mountinfo` and I can see for my procfs a line like this with the word shared in it:

```
170 24 0:4 / /home/paul/fs1 rw,relatime shared:146 - proc dummy rw
```

If I call `mount --make-private fs1`, the `shared:146` portion of the line goes away.

How does a shared mount differ from a private one?  A shared mount means that subsequent calls to `umount` and `mount` will propagate to other `--bind` mounted file systems.  In a private mount, those calls will not propagate.  To illustrate this, here is an example:

```
$ mkdir fs1 fs2 subd1
$ sudo mount --bind fs1 fs1
$ sudo mount --bind fs1 fs2
$ sudo mount --make-private fs1
$ mkdir fs1/sub_mount
$ ls fs2
sub_mount
# ^ Files are propagated between the mounts

$ sudo mount --bind subd1 fs1/sub_mount
$ touch subd1/hello
$ ls fs1/sub_mount
hello
$ ls fs2/sub_mount
$ # nothing
```

It turns out that `unshare` mounts all its new mounts as private by default (see [unshare(1)][unshare-1]).  Since the purpose of unshare is to isolate it from other things, this makes sense conceptually, but practically, I am having a hard time thinking of when a proc file system being shared would be a thing that mattered, but then again I do not know if there is more mounting that happens during the creation of a `proc` fs.


# Conclusion

This post was a bit of a random walk around `mount` and `unshare`, but I learned something during it, so I wanted to record it in case I find myself wondering about it later.


[unshare-source]:https://github.com/karelzak/util-linux/blob/master/sys-utils/unshare.c#L454-L455


[container-post]:{% post_url 2017-04-19-containers %}

[core-os-talk-slides]:https://speakerd.s3.amazonaws.com/presentations/eb9b416908c743f99c20d05d060209ae/coreos-fest-2017.pdf

[core-os-talk-video]:https://www.youtube.com/watch?v=wyqoi52k5jM

[bucket-repo]:https://github.com/kevindrosendahl/bucket

[man-page-numbers]:https://stackoverflow.com/questions/62936/what-does-the-number-in-parentheses-shown-after-unix-command-names-mean

[mount-8]:https://linux.die.net/man/8/mount
[mount-2]:https://linux.die.net/man/2/mount

[mount-existing-2]:https://unix.stackexchange.com/questions/198542/what-happens-when-you-mount-over-an-existing-folder-with-contents?rq=1

[mount-existing-1]:https://unix.stackexchange.com/questions/251090/why-does-mount-happen-over-an-existing-directory

[so-device-names]:https://superuser.com/questions/558156/what-does-dev-sda-for-linux-mean\
[wiki-fstab]:https://en.wikipedia.org/wiki/Fstab
[unshare-1]:http://man7.org/linux/man-pages/man1/unshare.1.html
