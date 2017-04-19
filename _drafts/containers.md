---
layout: post
title: "Containers - How do they work?"
---

# Motivation

I hear the word "container" in the software context a lot but I do not really know it means.  To quote [Tim Urban][wbw] ([again][first-post]), I get the "ugh it’s that icky term again nah go away" feeling.  I also definitely have some [magpie tendencies][magpie], so it was difficult for me to tell how much of the buzz was about shiny-ness and how much was immediately useful. ([This article][container-joke] captures how I felt any time containers came up in articles or Hacker News comments.) I decided it was time to roll up my sleeves and dive in.


# Introduction

My knowledge about containers started out basically with the Docker logo of a whale with a bunch of shipping containers on it.  As an analogy, shipping containers seem like they would be really nice in the software world. Shipping containers are stackable, isolated, easy to move around and flexible in terms of being useful on ships, trucks, and trains.  Having software in well defined boxes like that sounds since generally the software I work with requires a lot of setup to get the environment right, and it is often tough to run multiple projects if they need slightly different environments.  Additionally, moving something from my laptop to the server does not feel as simple conceptually as moving a shipping container from a boat to a train.

In the rest of this post, I'll try to evaluate how accurate that analogy is by walking through what goes into containers.  I'll start with a rough idea of the history (I'm sure I will get some things wrong, feel free to [let me know][twitter])

# History - How did we get here?

In [an interview][eric-brewer], Eric Brewer, a VP of infrastructure at Google, shares an interesting take on the origin of containers.  He says that Google was born in 1998 before "virtual machines" really existed.  This meant they were running directly on the hardware which is very different from the cloud computing VM's we see today.  In a pre-VM world, they needed ways to run more code on the same machine while also enforcing usage limits and isolation.

The features used to do this effectively existed in several forms, but the word "container" was not widely used.  As best I can tell, it wasn't until [Docker][docker-github] packaged up the different features in a user-friendly way that the usage of containers really took off.

To help understand why, I find it helpful to understand the gap containers fill.  On the one hand, I can have a single machine running a bunch of different code with everything all mixed together.  As I mentioned previously, this can be difficult to manage and move things around.  On the other end of the spectrum, there are virtual machines. Virtual machines running on the same physical machine are very isolated (e.g. my EC2 instance won't accidentally interact with other EC2 instances running on the same hardware.)  However, a virtual machine is fairly heavy-weight.  Stretching the shipping container analogy, if my physical hardware is the boat, running a virtual machine for each piece of my code would be like putting individual semi-trucks under all of my shipping containers.  That would add a lot of weight to the boat's cargo.  Similarly, running many virtual machines requires the physical hardware to do a lot of work.  Containers are targeted at a sweet spot of isolation without needing an entirely separate underlying operating system to support them.

# What makes containers work?

How do containers accomplish isolation without a separate underlying machine?  There have existed several isolation mechanisms over the years, but I will focus on a few I found in use today.

### cgroup

First, there are `cgroup`'s, short for "control group".  This came out of Google in 2008 from Paul Menage and Rohit Seth.  It was an addition to the Linux kernel that limits and accounts for the resources used by a group of processes.  That means I can put processes in a `cgroup` and have visibility into how much memory and CPU it uses as well as put limits on what % of the CPU those processes can use when there is contention.  The [man page][cgroup-manpage] and the [Arch linux wiki][arch-cgroup] both have more detail on how this works.

### namespaces

Next there are [linux namespaces][linux-namespace].  These are what provides the isolation on several different categories.  When I create a new "namespace", processes running in that namespace have the view that they are the only processes running.  For instance, in each PID namespace, there can processes with PID 1.  With a separate "user" namespace, the process can operate as root within the namespace without having root privileges outside the namespace.  Similarly, in a `net` namespace, a process can talk to `localhost:80` even if outside the namespace there is something already bound to that port.  A good run-down of the different namespaces with examples is in this [toptal article][namespace-article] (and the [man page][namespace-manpage] also has more information).

### Union mounted file systems

The last piece of the puzzle is sort of like namespacing for the filesystem.  Two competing versions of this technology are AUFS and OverlayFS.  These two filesystems allow for copy-on-write views of the filesystem.  This means that each container gets its own view of the filesystem and can share things like the python binary for instance. However, if they modify it, a copy is made that only they can see.  Here is [an article][aufs-example] with examples of using AUFS which illustrates how this works in practice.

# Conclusion

Hopefully this was a useful introduction into the world of containers.  In subsequent posts, I want to dive into the difference between the container engines Docker and [rkt][rkt] as well as dig into what is actually inside a container file.  I have ambitions to make a toy container engine, so stay tuned for that as well.

[magpie]:https://blog.codinghorror.com/the-magpie-developer/
[wbw]:http://waitbutwhy.com/2015/06/how-tesla-will-change-your-life.html
[first-post]:{% post_url 2016-05-09-starting-out %}
[container-joke]:https://circleci.com/blog/its-the-future/
[rkt]:https://coreos.com/rkt/
[last-post]:{% post_url 2017-03-27-google-sign-in %}
[twitter]:https://twitter.com/paulcarletonjr
[lxd]:https://linuxcontainers.org/lxd/
[lxc]:https://en.wikipedia.org/wiki/LXC
[cgroups]:https://en.wikipedia.org/wiki/Cgroups
[nice]:https://en.wikipedia.org/wiki/Nice_(Unix)
[dancers]:https://en.wikipedia.org/wiki/The_Legion_of_Extraordinary_Dancers
[eric-brewer]:https://medium.com/s-c-a-l-e/google-systems-guru-explains-why-containers-are-the-future-of-computing-87922af2cf95
[chroot]:https://en.wikipedia.org/wiki/Chroot
[python-lxc]:https://linuxcontainers.org/lxc/documentation/
[aufs-example]:http://www.thegeekstuff.com/2013/05/linux-aufs/
[linux-namespace]:https://en.wikipedia.org/wiki/Linux_namespaces
[namespace-article]:https://www.toptal.com/linux/separation-anxiety-isolating-your-system-with-linux-namespaces
[namespace-manpage]:http://man7.org/linux/man-pages/man7/namespaces.7.html
[cgroup-manpage]:https://www.kernel.org/doc/Documentation/cgroup-v1/cgroups.txt
[docker-github]:https://github.com/docker/docker
[systemd-wiki]:https://en.wikipedia.org/wiki/Systemd
