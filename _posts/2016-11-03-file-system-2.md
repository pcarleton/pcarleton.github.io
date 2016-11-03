---
layout: post
title: "Using strace"
---


In my [last post][1], I discussed what a file descriptor is and ran some commands to interact with the file descriptors associated with a shell process.  In this post, I'll look at how to use the `strace` tool to inspect the file activity of a process.


# History

When using a new tool, I find it worthwhile to read about how the tool came about.  `strace` has a [wikipedia page][2] that tells us it is a "system call tracer" (hence the name s-trace) originally written for SunOS by Paul Kranenburg (1991).  It was ported to Linux by Branko Lankester (1992) and then refactored by Richard Sladkey (1994).  The page also mentions a kernel feature called [ptrace][3].  `ptrace` is a system call (I'll have a post about the ins and outs of system calls soon) which allows one process to control another.  In the case of `strace`, we're not actively controlling the other process, but inspecting it's system calls.  Debuggers like `gdb` use `ptrace` in order to control the flow of a program.  It can also be used as a sandbox like in the [sydbox][4] project.  This project intercepts system calls from the process and decides whether to accept or reject them.

# Using `strace`

One way to use `strace` is to start a process with it.  For instance:

```
$ strace ls
```

This will dump a lot of output to the screen. If we want to inspect it for later, we can save it to an output file like this:

```
$ strace -o ls.txt ls
```

In a later post, I'll dig through what all the lines in that output are doing. For now, I'll just appreciate how much goes into a simple call to `ls`.

We can also look at a running process's system call with the "-p" option.  For instance, when I run `strace` from one terminal with the PID of a separate shell process and then type "ab" into that process, I get the following:

```
$ strace -p 3227
Process 3227 attached
read(10, "a", 1)                        = 1
rt_sigprocmask(SIG_BLOCK, [WINCH], [], 8) = 0
write(10, "a", 1)                       = 1
rt_sigprocmask(SIG_UNBLOCK, [WINCH], [WINCH], 8) = 0
read(10, "b", 1)                        = 1
rt_sigprocmask(SIG_BLOCK, [WINCH], [], 8) = 0
write(10, "\10ab", 3)                   = 3
rt_sigprocmask(SIG_UNBLOCK, [WINCH], [WINCH], 8) = 0
read(10,
```

A couple interesting things come out here.  First, the process is separately reading in the input and then writing it back out.  Also, it writes out both "ab" rather than just "b" the second time around.  However, if we type "c" after, it just writes "c".  This seems to be true of the first two characters.  Finally, it is both reading and writing from [file descriptor][1] 10.  I would expect it to be reading from STDIN (fd 0) and writing to STDOUT (fd 1), however we can see from inspecting the process's fd directory, these all point to the same place: `/dev/pts/2`

```
ls -la /proc/3227/fd
total 0
dr-x------ 2 paul paul  0 Nov  3 07:26 .
dr-xr-xr-x 9 paul paul  0 Nov  3 07:26 ..
lrwx------ 1 paul paul 64 Nov  3 07:26 0 -> /dev/pts/2
lrwx------ 1 paul paul 64 Nov  3 07:27 1 -> /dev/pts/2
lrwx------ 1 paul paul 64 Nov  3 07:35 10 -> /dev/pts/2
lrwx------ 1 paul paul 64 Nov  3 07:26 2 -> /dev/pts/2
```
They all point to "pseudo terminal" #2.  (For a brief explanation of pseudo terminals see [this stack exchange post][5]).

One last thing on this I would like to point out are those calls to `rt_sigprocmask`.  Looking at the [man page][7] for `rt_sigprocmask`, we can see it is used for blocking or unblocking the delivery of signals to a thread.  This makes sense based on the arguments `SIG_BLOCK` and `SIG_UNBLOCK`.  The next question then is what is the `WINCH` signal?  Here is a list of [signals][6] which says that the `WINCH` signal is the " Window resize signal" (short for Window Change).  This means that in between reading and writing, the shell process ignores any resizes of the window.  This likely because a resize would change where we would want to write the character out, so we want to wait until we're finished righting before re-writing everything for the new window size.  We can observe a `WINCH` signal in action by resizing the window while strace is running.

# Conclusion

Hopefully that was a useful brief dive into `strace`.  For a more in depth dive, checkout [this walkthrough][7] of determining if a command line tool accepts wildcards.

[1]:{% post_url 2016-10-26-file-system-1 %}
[2]:https://en.wikipedia.org/wiki/Strace
[3]:https://en.wikipedia.org/wiki/Ptrace#cite_note-PRoot-2
[4]:http://freecode.com/projects/sydbox
[5]:https://linux.die.net/man/2/rt_sigprocmask
[6]:http://man7.org/linux/man-pages/man7/signal.7.html
[7]:http://events.linuxfoundation.org/sites/events/files/slides/lce-2015-strace-bash-en.pdf
