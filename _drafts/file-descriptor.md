---
layout: post
title: "What's a file descriptor?"
---


What is a file descriptor?


Wikipedia link, maybe an image from wikipedia.

Number representing an entry in the file table which says what mode the file is in.

Separately there is an inode table with an entry in it. (What is an inode?)

Using strace to see what file descriptors a process opens and what system calls it uses

Link to dtrace explanation for mac users (and stackoverflow article about disabling the rootless protection)


Try to snoop on the file-descriptor for a terminal, is that even possible?

Also hint at future post about making system calls like the ones you need to get a file descriptor.

File descriptors for sockets

Listed in /proc/<pid>/fd

Future post about pipes? Or maybe it is enough to fit in this post.

Where is the table stored? In RAM i guess?

What is "ls" doing?

Talk about the goal of the blog being to go a little bit further down the layers of abstraction than I normally would.  How is the filesystem organized? And where is it stored? Is it a file somewhere?

Going to try snooping on a file descriptor.

With strace, you get the write syscalls with the contents, so you can see what's being written. Kind of roundabout.

https://en.wikipedia.org/wiki/Strace

http://tldp.org/LDP/Linux-Filesystem-Hierarchy/html/proc.html


Still want to know what an inode is doing.

Found out why that ROS stuff was in there.  It was something to do with my zsh profile that made it look in the ros libs.  It has to do with the LD libary path I think.

What is the LD Library path?  Maybe that's for another post.
