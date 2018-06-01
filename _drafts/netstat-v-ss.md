---
layout: post
title: "TIL: The difference between netstat and ss"
---

### Motivation

In this post, I'm going to go over two tools for inspecting the state of sockets and  why people recommend `ss` instead of `netstat`for inspecting socket states.   This is not going to be a 12 ways to inspect socket states article, there are lots of those.


### What is a socket?

A socket is a Linux file descriptor for communicating with the network. In Linux, they say everything is a file.  In this case, you can treat a socket like a file that writes to the network instead of writing to a disk.  Sockets come in different flavors for TCP vs. UDP.  For more on sockets, checkout these links:

* [The Linux Programming Interface](http://man7.org/tlpi/) (this is really great)
* [Beej's Network Programming Guide](http://beej.us/guide/bgnet/html/multi/theory.html) (not quite TLPI, but available for free)



### Why would I care about sockets?

Sockets can be in a bunch of different states (listed in [this snippet of ss](https://github.com/sivasankariit/iproute2/blob/master/misc/ss.c#L80-L92)) which can be useful to answer questions like:

* Is my server process actually listening on the port I think it is?
* Is it listening on the loopback interface (`127.0.0.1`), or could someone on my network connect to it (`*` or `0.0.0.0`)?  (This can be bad if you're developing something and running a database that anyone in the coffee shop can make requests to)
* What processes are listening on what ports in general?

These questions pertain to sockets that are listening on a host, but there are also various connected states a socket can be in.  [High Performance Browser Networking](https://hpbn.co/building-blocks-of-tcp/#three-way-handshake)'s TCP overview if great for some explanation of the other states (or again [TLPI](http://man7.org/tlpi/) is excellent)



### How do I inspect the state of sockets on my machine?

The two ways I'll talk about doing this are through `netstat` and `ss`.  Both are command line utilities that are installed on almost all Linux machines.   Before I get into why I would use one or the other, let's look into where they get their information.



#### Procfs

The [Procfs](https://en.wikipedia.org/wiki/Procfs) is a file system that Linux exposes that is like a peek into kernel memory.  It lives in `/proc` and it exposes information about TCP and UDP sockets at `/proc/net/tcp` and `/proc/net/udp`.  If I `cat` either of those I'll get some inscrutable output.



#### Netlink

The other source of information is called the `netlink` protocol.  In this case you open a socket (a `SOCK_RAW` with `AF_NETLINK` as seen [here](https://github.com/sivasankariit/iproute2/blob/1179ab033c31d2c67f406be5bcd5e4c0685855fe/misc/ss.c#L1650)).  I can then send requests on that socket for information about other sockets (pretty meta).  I have a lot to learn still about `netlink`, but here are some things I found:

* [`libnl`documentation](https://www.infradead.org/~tgr/libnl/doc/core.html) which is a library for interacting with `netlink` sockets.
* [RFC 3594](https://tools.ietf.org/html/rfc3549) -- Linux Netlink as an IP Services Protocol



### Netstat vs. ss

Now that we understand the sources of information, why would we prefer one tool over the other?  `netstat` gets its information from `/proc/net` directly.  It parses the file and prints out information based on it.

`ss` on the other hand was written more recently to use the `netlink` API (it will fall back to `proc/net` if netlink is unavailable).  The information is essentially the same from what I've seen, but here are some arguments for why to use `ss`

* Supposedly it's faster (I just read that a lot, I don't find `netstat` to be that slow to be honest)
* Netlink exposes more TCP states (again I mostly look for `LISTEN` so that's not a huge selling point)
* It has better default argument behavior

None of these are a huge homerun, which is why I expect a lot of people still use `netstat`.  It's also likely that `netstat` is installed more places. For instance on my macbook, it has `netstat` but not `ss`.

The last one is a little more compelling for me personally.  `netstat` by default will try to resolve IP addresses which really slows it down and also opens a bunch of new UDP sockets, which might clutter the picture if you're investigating something.  `netstat -n` stops this behavior, but `ss` has that on by default.

Another reason I prefer `ss` is that the code is much nicer to read!

### Open Questions

Here are some things I am still wondering

* What are the states that netlink supports that `netstat` won't show? Are they states I actually care about?
* Where does `lsof -i` fit in with all of this? Why would I choose that over `ss`?
* What other users are there for netlink?
* Can I use netlink to poll for any new UDP connection? (I've been wanting to do this to figure out what process is sending UDP packets to a particular IP address, which is easy with TCP but hard with UDP)
