---
layout: post
title: "How does cron work?"
---

# TL;DR

The `cron` utility is a daemon that wakes up every minute to check if needs to run a job.

# The Story of Cron

I found myself wondering recently how `cron` works (`cron` is a utility for scheduling tasks on Linux. For a good intro on how to use it, see this [blog post][7]).  Is `cron` doing something fancy to schedule these tasks? Or is it just constantly checking if it has something to do?  It turns out that over its history, it has done both.

The [Wikipedia page][2] on `cron` gives a thorough history which I'll summarize.  It was originally written by [Ken Thompson][4] in the late 70's, and it did a simple check every minute.  When mainframes with 100s of users tried using `cron`, this check for every user's `cron` jobs every minute required too many resources.  Keith Williamson wrote a new version (SysV cron) in 1979 which borrowed ideas from a paper on "discrete event simulators" that had a queue.  It put every user's jobs on the queue, and then slept until it was time to run the next job.  It also woke up every 30 minutes to refresh the queue.  This version was able to handle machines with many users with an acceptable amount of resources.

The modern implementation of `cron` was written by [Paul Vixie][5] in 1987 and went back to checking every minute presumably because resources were no longer as scarce. From Debian's cron [man page][3]:

> cron  then  wakes up every minute, examining all stored crontabs, checking each command to see if it should be run in the current minute.

As to what differentiates Vixie Cron from SysV cron, we get a hint from the Red-Hat package manager [docs][7]:

> Vixie cron adds better security and more powerful configuration options to the standard version of cron.


# Conclusion

I thought there may be a complicated system underlying `cron`.  However, as it turns out with most things when you get to the bottom of them, it is pretty simple.


[1]:http://stackoverflow.com/questions/3982957/how-does-cron-internally-schedule-jobs
[2]:https://en.wikipedia.org/wiki/Cron
[3]:http://www.unix.com/man-page/debian/8/cron/
[4]:https://en.wikipedia.org/wiki/Ken_Thompson
[5]:https://en.wikipedia.org/wiki/Paul_Vixie
[6]:https://wiki.gentoo.org/wiki/Cron#vixie-cron
[7]:http://troy.jdmz.net/cron/
