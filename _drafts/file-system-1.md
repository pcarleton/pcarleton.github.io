---
layout: post
title: "What's a file descriptor?"
---

# Getting out of the Comfort Zone

When I [started this blog][1], I decided I wanted to do deep dives on topics in order to provide value.  I have realized that by asking questions about things I use everyday can lead to all sorts of new knowledge if I stretch past my comfort zone.  Digging deep enough to get to the bottom of a topic takes work, but the stuff you find on the way has some serious staying power.  One of the great things about Linux is that it is relatively easy to dig down to the bottom of a particular question since it's all open source.

Continuing in that same vein, I have heard the word "file descriptor" a couple times in the past month.  It's one of those phrases where I can guess what it means and move on ("It's something to do with files, right? I read and write files all the time, moving on..."), or I can peel back the layers of abstraction to get a better sense of what's really going on.  So the next couple posts I will be taking a dive into the Linux file system.

# What is a file descriptor?

First, I'll be looking at file descriptors.  What is a file descriptor? It sounds like some kind of description of a file.  The [wikipedia page][2] is pretty helpful here in giving us an introduction.  A file descriptor is an abstraction around files.  It an integer that serves as a pointer to any "input/output" resource.  This means it could represent a file on disk, a network socket, or a unix pipe.  Each process gets at least 3 file descriptors, 0 - Standard In, 1 - Standard Out and 2 Standard Error.  This is why sometimes you see a `2>&1` in a shell script which is redirecting Standard Error to Standard Out.

These numbers index into each process's individual file descriptor table.  So when you start a new process, you get a new file descriptor table, and its contents are available in `/proc/<pid>/fd`.  These act just like regular files too.  For instance, try the following:

```
# Open 2 separate terminal windows, and find their process ID's
% ps aux | grep /usr/bin/zsh
paul     14181  0.0  0.1  47408  5056 pts/1    Ss   Oct25   0:01 /usr/bin/zsh
paul     15674  0.0  0.1  47324  4396 pts/2    Ss+  Oct25   0:01 /usr/bin/zsh

# Echo something into the Standard Error of the other shell process
% echo "hi!" >/process/15674/fd/2
# You should shee "hi!" pop up in the other terminal!
```

Additionally, if you do something like `tail -f /proc/<pid>/fd/0` in an attempt to read all of Standard In for one of the terminals, you'll see that all (or most) of your input in that terminal gets sucked away and does not appear on screen.  Interestingly, none of it shows up in the output of the `tail` call (I'm not sure why, but stay tuned and I'll try to get an explanation.)

So now we now the very basics of what a file descriptor is and where it lives.  In the next post, I'll talk about how to use `strace` to snoop on a process's file descriptor activity without squashing it like we did with `tail`.
