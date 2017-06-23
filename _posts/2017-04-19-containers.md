---
layout: post
title: "Containers - How do they work?"
---

# Motivation

I hear the word "container" in the software context a lot but I do not really know it means.  To quote [Tim Urban][wbw] ([again][first-post]), I get the "ugh it’s that icky term again nah go away" feeling.  I also definitely have some [magpie tendencies][magpie], so it was difficult for me to tell how much of the buzz was about shiny-ness and how much was immediately useful. ([This article][container-joke] captures how I felt any time containers came up in articles or Hacker News comments.) I decided it was time to roll up my sleeves and dive in.


# Introduction

I'll start with a statement that I am using to describe containers and then I will break down what it means:

Containers are a lightweight way to run code with CPU, memory and file-system isolation using Linux kernel tools.

## The Word "Container"

First, let's talk about the word "container".  My knowledge about containers started out with the Docker logo of a whale with a bunch of shipping containers on it.

![Docker logo][docker-logo]{:height="120px" style="float: left;margin-right: 7px;margin-top: 7px;"}

As an analogy, shipping containers seem like they would be really nice in the software world. Shipping containers are stackable, isolated, easy to move around and flexible in terms of being useful on ships, trucks, and trains.  Having software in well defined boxes like that sounds since generally the software I work with requires a lot of setup to get the environment right, and it is often tough to run multiple projects if they need slightly different environments.  Additionally, moving something from my laptop to the server does not feel as simple conceptually as moving a shipping container from a boat to a train.

## Isolation

The next topic to cover is isolation.  While the shipping container analogy seems nice in theory, what are the actual reasons we would want resource isolation?

The first is security.  Some software wants to run as root on your machine, but you may have sensitive stuff on your machine that you don't want to give it access to.  Isolation in this sense allows the code to act as if it is root, but without allowing it to wreck havoc on the rest of your machine.

The second is reproducibility.  When code is isolated to a sort of "clean-room" environment, it is much easier to get reproducible output than when it is interacting with arbitrary other resources on the machine.  For instance, some code interacts with system installed libraries, and if different machines have different versions, they could fail in different ways.  Isolation reduces this problem by reducing the number of things that are different between runs at different times and on different machines.

The third is resource scarcity.  A machine only has so much CPU and memory.  Software has bugs, and sometimes that means a process will have runaway CPU or memory requirements.  Without isolation, this could rob other processes of resources, and cause them or even the whole operating system to crash.  With isolation, the effect of the runaway process can be limited to just causing itself to crash.

## "Lightweight"

The word lightweight in a vacuum does not mean much.  Light in comparison to what?

With the goal of isolating two processes, the heavyweight end of the spectrum is a fully separate machine.  This is heavy because it requires maintaining entirely separate hardware including power requirements, components, and physical space.  This provides very good isolation since the only way the two components could interact would be over a network connection between them.

In the medium range of the spectrum is a virtual machine.  A VM is simulating hardware in software.  This means that starting up a new VM requires simulating the entire process of booting a real machine complete with its own separate operating system.  In addition to the slow boot process, a chunk of the underlying hardware's resources has to go to simulating all the necessary aspects of the hardware. This is lighter weight than having totally separate hardware, and still offers fairly solid isolation.  For instance, it is rare that you notice any other hosts running when you are running code on an Amazon EC2 instance (the exception being when another host is being particularly resource hungry).

On the lightweight end of the spectrum, we have containers.  Since a container does not have boot its own operating system running, it can start up relatively quickly.  It also does not need as many resources as running virtual hardware.  It does provide slightly weaker isolation guarantees when compared to the other two options.  This is a double edged sword because sometimes you want less isolation so more things can be shared.

## Linux Kernel Features

The underlying pieces that make containers work are all part of the Linux kernel.  Tools like Docker are like opinionated wrappers around these kernel features.  To see what the different isolation mechanisms there are, you can run `ls /proc/<pid>/ns`, where `ns` stands for namespaces.  On my machine, I see:

```
cgroup  ipc  mnt  net  pid  user  uts
```

Briefly those are "control groups", "inter process communication", "mount" like file system mounts, "network", "process ID", "user", "uts" allows different hostname and domain names.  I'll take a look at a few of these below.

### cgroup

First, there are `cgroup`'s, short for "control group".  This came out of Google in 2008 from Paul Menage and Rohit Seth.  It was an addition to the Linux kernel that limits and accounts for the resources used by a group of processes.  That means I can put processes in a `cgroup` and have visibility into how much memory and CPU it uses as well as put limits on what % of the CPU those processes can use when there is contention.  The [man page][cgroup-manpage] and the [Arch linux wiki][arch-cgroup] both have more detail on how this works.

### namespaces: pid, user, net

Next there are [linux namespaces][linux-namespace].  These are what provides the isolation on several different categories.  When I create a new "namespace", processes running in that namespace have the view that they are the only processes running.  For instance, in each PID namespace, there can processes with PID 1.  With a separate "user" namespace, the process can operate as root within the namespace without having root privileges outside the namespace.  Similarly, in a `net` namespace, a process can talk to `localhost:80` even if outside the namespace there is something already bound to that port.  A good run-down of the different namespaces with examples is in this [toptal article][namespace-article] (and the [man page][namespace-manpage] also has more information).

### Union mounted file systems

The last piece of the puzzle is sort of like namespacing for the filesystem.  Two competing versions of this technology are AUFS and OverlayFS.  These two filesystems allow for copy-on-write views of the filesystem.  This means that each container gets its own view of the filesystem and can share things like the python binary for instance. However, if they modify it, a copy is made that only they can see.  Here is [an article][aufs-example] with examples of using AUFS which illustrates how this works in practice.

# Conclusion

That brings us back to our summary:

Containers are a lightweight way to run code with CPU, memory and file-system isolation using Linux kernel tools.

We talked about what the word "container" means in an analogy.  We talked about what resource isolation means and why we would want it.  We talked about what containers are lightweight in comparison to, and we talked about some of the individual Linux kernel features that make containers possible.

In subsequent posts, I want to dive into the difference between the container engines Docker and [rkt][rkt] as well as dig into what is actually inside a container file.  I have ambitions to make a toy container run time, so stay tuned for that as well.

(Note: this post was heavily revised on 6/23/2017)

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
[docker-logo]:https://upload.wikimedia.org/wikipedia/commons/7/79/Docker_%28container_engine%29_logo.png
