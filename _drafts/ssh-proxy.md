---
layout: post
title: "SSH ProxyCommand with Netcat"
---

# SSH ProxyCommand with Netcat

I found myself looking at articles regarding SSH ProxyCommand's this week.  For work, I needed to SSH to one machine, change user, and then SSH to another machine.  There are a [number][1] of [useful][2] [articles][3] that give commands to make this kind of thing work.  I'll go through a couple of the commands mentioned in those articles, describe what they do, and then dig a little deeper into how they do it.

Here is a common one:

```
Host machineB
  ProxyCommand ssh machineA nc %h %p
```

This is connecting to machineB using machineA as a "proxy".  That means all the traffic from your computer to machineB and from machineB to your computer passes through machineA.  This is useful if machineA is accessible from the public internet, but machineB is not.  After putting those lines in your `~/.ssh/config`, you can type `ssh machineB` and it will do the proxying for you behind the scenes.

Now what is that command actually doing?  It first opens an ssh connection to machineA, then on that machine calls the `nc` (netcat) utility.  The arguments to this are `%h %p` which get expanded to the provided host and port.  In the case of us calling `ssh machineB`, these arguments would become `machineB 22` since 22 is the default port for SSH.  Using these template parameters is useful if you want to do pattern matching in your config file.  For instance, instead of `Host machineB`, you could do `Host *.yourcompany.com`, and then you wouldn't need a separate set of commands for each of your machines.  (Sidenote: you can also exclude certain hosts, so you might do something like `Host *.yourcompany.com !firsthop.yourcompany.com` if `firsthop` is the machine that you can connect to.)

Netcat is a networking utility that can do lots of neat things.  Here's a snippet from the manpage (on Debian) describing what it's doing in this scenario:
```
In the simplest usage, "nc host port" creates a TCP connection to the given port on the given target host.  Your standard input is then sent to the host, and anything that comes back across the  connection  is  sent  to  your standard output.
```

So the `ProxyCommand` is creating a process which has it's STDIN and STDOUT hooked up to a TCP connection on `machineB`.  The `ssh` client then starts sending data over that STDIN and receiving it via the STDOUT to establish it's connection.  You can see this in action by using regular `cat` instead of netcat in a `ProxyCommand`.  If we run `tee log.txt | cat` by itself, we can type lines and have them echoed back to us while also having the lines we enter saved to a file.  Using it as a ProxyCommand, we can see what `ssh` sends to an ssh server while also echoing it back to the process like this:

```
$ ssh -o ProxyCommand='tee log.txt | cat' anyhost
Disconnecting: Protocol error: expected packet type 31, got 30
```

As you can see, it fails to make a connection.  It sent a packet type 30, and received the same type back while it expected to get type 31.  Our `log.txt` looks like this:

```
SSH-2.0-OpenSSH_6.7p1 Debian-5+deb8u3
����O_+�O�D�$�curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384...
```

The first line declares it's protocol type, and the next lines appear to be describing something about encryption.  I looked at the [RFC for SSH][rfc], but I did not see anything about packet type 30 vs 31.  I will leave that investigation for another day.

[1]:http://sshmenu.sourceforge.net/articles/transparent-mulithop.html
[2]:http://www.cyberciti.biz/faq/linux-unix-ssh-proxycommand-passing-through-one-host-gateway-server/
[3]:https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Proxies_and_Jump_Hosts
[rfc]:https://tools.ietf.org/html/rfc4253#section-4.1
