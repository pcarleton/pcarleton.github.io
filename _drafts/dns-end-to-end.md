---
layout: post
title: "DNS End to End"
---

# Domain Name System: End to End

We all interact with the Domain Name System (DNS) every day.  Every time we load a web page, or click a link, our software relies on DNS to figure out what address to send requests to. In this post, I will dig into how DNS works by tracing through the machinery that happens behind the scenes.


# What is the purpose of DNS?

It is first worth calling out what DNS is and why anybody should care about it.  A good place to start is [Domain Names - Implementation and Specification RFC1035](https://www.ietf.org/rfc/rfc1035.txt).  In the introduction, it says:

> The goal of domain names is to provide a mechanism for naming resources
in such a way that the names are usable in different hosts, networks,
protocol families, internets, and administrative organizations.

Domain names are supposed to be a convenient, general purpose way to refer to resources.  Convenient in this case most likely means relative to referring things to an actual host address. Practically, this means DNS turns "google.com" into an IP address like "216.58.192.14" [^1]. This provides a number of benefits.  "google.com" is much easier to remember than a series of 4 numbers.  Also, a web service can change which host address it wants to use without end users even noticing[^2].

# DNS Resolution "by hand"

Now that we know the goal of DNS, I want to demonstrate how DNS works by crafting DNS queries using `dig`.  I will ignore a lot of pieces of the system for this illustration (like caching and recursive resolution). I will try to resolve "pcarleton.com"

## Root Domain

In order to figure out what IP address is serving "pcarleton.com.", I can work my way down the hierachy.  There are servers at each level of the hierarchy that can tell me where to find information for the level below. The domain name "pcarleton.com" becomes `["", "com", "pcarletion"]` in the hierarchy where the empty string is the implied "root" domain[^4].

A name server at the root level can tell me about what nameservers are responsible for each Top Level Domain (TLD) like ".com", ".net" etc.  In order to query a root server, I need to know one of its IP addresses.  I can pick one from the [IANA website](https://www.iana.org/domains/root/servers), so I'll pick `a.root-servers.net` with IP address `198.41.0.4`[^5]

I can then query this root server for what name servers are responsible for `pcarleton.com` via `dig -t ns pcarleton.com @198.41.0.4`.  This gives me a list of servers which are responsible for the `.com` TLD.

## .Com Domain

The root server gave a list of `.com` nameservers (and IP addresses) that look like `a.gtld-servers.net` with the letters A through M.  For instance, `a.gtld-servers.net` has address: `192.5.6.30`.  I can then query this address to see which `.com` name server is responsible for the `pcarleton` domain name with `dig -t ns pcarleton.com @192.5.6.30`.

## Namecheap DNS servers

This query reveals that `dns1.registrar-servers.net` has information for `pcarleton.com` (and has IP `216.87.155.33`).  This name server is the one run by Namecheap which is where I registered `pcarleton.com`.  If we query this one with `dig pcarleton.com @216.87.155.33`, we get the IP address of this site which we can then use.

##  Making Changes

This example demonstrated how to interact with the DNS system, but it did not show how changes to that information would propagate.  To understand that, let's look back at what data was required:

<img src="{{ site.url }}/assets/dns/data.png" width="600px">


Changes to the list of root IP's should never happen.  If it did, it would require pushing a file listing the new IP's to all running DNS servers by system admins all over the world.

Changes of the mapping of TLD's to authoritative name servers does not happen frequently, but is administered by ICANN. It will be communicated to all the root servers which will update their records and serve them to requests.

Changes to the TLD server's mapping (the Registry) changes more frequently.  Every time a new domain name is registered, the TLD'd servers need to know which domain name servers have the required IP addresses. A Registrar makes a request to the "Registry" to update these records.

Changes to the final DNS servers mapping of domain name to IP address can happen much more frequently since it only needs to be updated in the two DNS servers listed. I can change this by making a request to Namecheap.

# Reality Check

This example showed how some of the pieces of the DNS system work, but it is not typically how a DNS request goes.  In reality, the client usually makes a request to a DNS server[^5] which has a lot of information cached (like the .com TLD servers) and will make requests to the appropriate servers rather than telling the client which nameservers to look at[^6].


# Further Information

Here's a list of resources to look for further information about DNS:


[Internet Numbers Registry System - RFC 7010](https://tools.ietf.org/html/rfc7020)
[List of Local Internet Registries](https://www.ripe.net/membership/indices/US.html) (This who Namecheap could contract with to update the nameserver mapping)

[IANA's role - RFC 1591](https://tools.ietf.org/html/rfc1591)

[Jon Postel (the original IANA)'s RFC Eulogy](https://tools.ietf.org/html/rfc2468)

[Jon Postel root swapping incident](http://songbird.com/pab/mail/0472.html)

[Argument that ICANN's authority is illegal](http://osaka.law.miami.edu/~froomkin/articles/icann-body.htm#H1N5)

[Unified Domain-name Dispute Resolution Policy](https://www.icann.org/resources/pages/help/dndr/udrp-en)

[Email Template to update IP for a TLD name server](https://www.iana.org/domains/root/tld-change-template.txt)

[ARIN](https://www.arin.net/about_us/overview.html)

[RIR](https://en.wikipedia.org/wiki/Regional_Internet_registry)

[^1]: You can test this out locally by running `dig google.com` from the command line. If you put this IP address into your browser, it will load google.com.

[^2]: It is not difficult to imagine how a bad actor could cause trouble with this too. If they were to compromise DNS information, they could cause clients to send information to a location where they can intercept it.  For this post, I won't get into that, and I will pretend like everybody on the internet are behaving nicely.

[^3]: "pcarleton.com" is implicitly re-written as "pcarleton.com." where the root domain is the empty string at the end.

[^4]: Ordinary websites do not load "iana.org" in order to figure out root servers.  These 13 name servers are like internet constants in that their IP addresses never change.  Domain name servers usually have these IP addreses in a file somewhere.

[^5]: One such server is [Google Public DNS](https://en.wikipedia.org/wiki/Google_Public_DNS) which resides at IP address `8.8.8.8`.  


[^6]: This is assuming that the request is "recursive".  A recursive request asks that the server make requests to other DNS servers if it doesn't have the answer.  An "iterative" request by contrast asks that the server tell the client what server will have the answer if it does not have the answer.  A DNS server gets to decide whether it supports "recursive" requests.

