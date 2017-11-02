---
layout: post
title: "Dumb Rusty Toolbox: Dig (DNS Client)"
---


# Introduction

I have started a project that I am calling the Dumb Rusty Toolbox (drt for short) where I implement very dumbed-down versions of common command line utilities.

In this post, I'm going to describe ddig (dumb dig) which is a very rudimentary DNS client written in Rust.


# Goals

Answer these questions:

* What is the purpose of DNS?
* How does DNS work at a high level?
* How does DNS work at the protocol level?


# What is the purpose of DNS?

The best source for this question is the [Domain Names - Implementation and Specification RFC1035](https://www.ietf.org/rfc/rfc1035.txt).  In the introduction, it says:

> The goal of domain names is to provide a mechanism for naming resources
in such a way that the names are usable in different hosts, networks,
protocol families, internets, and administrative organizations.

> From the user's point of view, domain names are useful as arguments to a
local agent, called a resolver, which retrieves information associated
with the domain name.  Thus a user might ask for the host address or
mail information associated with a particular domain name.  To enable
the user to request a particular type of information, an appropriate
query type is passed to the resolver with the domain name.  To the user,
the domain tree is a single information space; the resolver is
responsible for hiding the distribution of data among name servers from
the user.

Domain names are supposed to be a convenient, general purpose way to refer to resources.  Convenient in this case most likely means relative to referring things to an actual IP address (or other identifier).  If a user had to remember the IP address for "Google" for instance, that would not be very convenient.


# What are the big components of DNS?

# How does authority work?

As the RFC states, there is a "local agent resolver".  This resolver communicates to a nameserver to retrieve information associated with a domain name.  The nameserver might be authoritative for a particular domain name, it might have cached information from another server that is authoritative, or it might have the address of a nameserver that has the information.

An interesting question here is how does a nameserver become authoritative.  For example, if I booted up a server and started serving DNS queries saying that I am the authority for "\*.paul", how would I let other name servers know I'm authoritative on some domain? How does the system prevent another person from claiming they are authoritative for that domain?

I tried to get at this question by querying DNS for the authority for the root domain (i.e. ".", more on this later).

This query looks like `dig -t ns "." @8.8.8.8` (that's type NameServer, domain name "." and 8.8.8.8 is Google's DNS servers) and returns a list of `{a-m}.gtld-servers.net` (13 servers in total). If you go to [IANA's website](https://www.iana.org/domains/root/servers), they list who owns each of those top level name servers.  (IANA is an affiliate if ICANN, more on that later).  They also list who is the TLD manager for every TLD in their [root database](https://www.iana.org/domains/root/db).

They provide a web interface and an [email template](https://www.iana.org/domains/root/tld-change-template.txt) for making changes to root name servers (for example, if you want a new IP address to be authoritative for a TLD).


[ARIN](https://www.arin.net/about_us/overview.html)
[RIR](https://en.wikipedia.org/wiki/Regional_Internet_registry)

Internet Numbers Registry System
[RFC 7020](https://tools.ietf.org/html/rfc7020) 2013
obsoletes [RFC 2050](https://tools.ietf.org/html/rfc2050) 1996
obsoletes [RFC 1466](https://tools.ietf.org/html/rfc1466) 
obsoletes [RFC 1366](https://tools.ietf.org/html/rfc1366) 1992


[List of LIR's in US](https://www.ripe.net/membership/indices/US.html)

IANA's role is laid out in [RFC 1591](https://tools.ietf.org/html/rfc1591) where it states some guidelines about selection of managers for TLD's.  It also states that the day-to-day activity is handled by the Internet Registry (IR).







TODO: Explain IANA and ICANN
TODO: Introduce the hierachy of domain names and local authority.
TODO: Answer the "." question

The first question I wanted to answer is what is 
