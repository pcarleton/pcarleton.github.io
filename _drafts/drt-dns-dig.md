---
layout: post
title: "Dumb dig clone in Rust"
---


# Why write a DNS client

When I started digging in to DNS, I thought it would be interesting to try implementing a very simple DNS client similar to `dig`, but using Rust. My goal was to better understand DNS at the protocol level.  I wanted to know what bytes were being sent and received to drive the system.

The final result is in [this repo][dns_repo]).  In the rest of the post, I'll detail what I found out along the way.

# The Protocol

The DNS protocol is detailed in [Domain Names - Implementation and Specification RFC1035](https://www.ietf.org/rfc/rfc1035.txt). Section "4.1 MESSAGES - Format" details the bytes of the protocol.  Here are the ascii diagrams I found useful:


### Message
```

    +---------------------+
    |        Header       |
    +---------------------+
    |       Question      | the question for the name server
    +---------------------+
    |        Answer       | RRs answering the question
    +---------------------+
    |      Authority      | RRs pointing toward an authority
    +---------------------+
    |      Additional     | RRs holding additional information
    +---------------------+
```

### Header

```

                                    1  1  1  1  1  1
      0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                      ID                       |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |QR|   Opcode  |AA|TC|RD|RA|   Z    |   RCODE   |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                    QDCOUNT                    |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                    ANCOUNT                    |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                    NSCOUNT                    |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                    ARCOUNT                    |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
```


### Question Section

```
                                    1  1  1  1  1  1
      0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                                               |
    /                     QNAME                     /
    /                                               /
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                     QTYPE                     |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                     QCLASS                    |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
```




### Resource Record (answer)

```
                                    1  1  1  1  1  1
      0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                                               |
    /                                               /
    /                      NAME                     /
    |                                               |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                      TYPE                     |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                     CLASS                     |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                      TTL                      |
    |                                               |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                   RDLENGTH                    |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--|
    /                     RDATA                     /
    /                                               /
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
```

# Implementation

I started out by "hand" building a DNS query for google.com with hard-coded values to ensure my tool was working.  Once I had this working, I started taking input from the command line.

I am relatively new to Rust, so I was initially trying to put everying on the stack and have nothign on the heap.  While a nice idea in theory, accepting input from the command line, or parsing dynamically sized elements from a server response make this pretty impractical.  I considered having a really large array and saying anything above that size wasn't supported, but ergonomically this didn't make the code any easier to work with.

Around this time, I came across [TRust DNS](https://github.com/bluejekyll/trust-dns), "A Rust based DNS client and server, built to be safe and secure from the ground up."  I took some inspiration from the message parsing code there, specically the `bindecoder` idea.

The end result was a client that could make a very simple DNS query and spit back the response.

# Weird and Interesting Things I came across

* Compression.  The protocol uses a bespoke method of compression where it will refer to resource names that have previously been seen.  It starts with `11` in the leading bits which distinguishes it from an ordinary label because an ordinary label starts with a length byte, and `11` as the leading bytes would make the label too long (they're limited to 63 bytes).

* Bash has a special builtin tool for opening UDP and TCP sockets -- I used this to sanity check my tool very early on (see `hello_udp.sh` in [the repo][dns_repo]).  See [this post](http://xmodulo.com/tcp-udp-socket-bash-shell.html) for more details on how it works.  (Notably, these builtins are not present in Zsh which threw me off).

* [Unified Domain-name Dispute Resolution Policy](https://www.icann.org/resources/pages/help/dndr/udrp-en)
* [Postel DNS Root incident](http://songbird.com/pab/mail/0472.html)
* [Is ICANN legal?](http://osaka.law.miami.edu/~froomkin/articles/icann-body.htm#H1N5)
* [TLD Change email template](https://www.iana.org/domains/root/tld-change-template.txt)


# Unanswered Questions and future work

* Who assigns new public IP addresses? And who maintains the existing mappings?
* Is there a super nice CLI library like Cobra Commander for Go in Rust?
* Recently, I wanted to know which process was issuing a DNS query against a DNS host I needed to shut-down, but I couldn't seem to do it.  It would be nice if there were a tool to trace particular traffic to an individual process.
* Writing this super simple version of a common command line tool taught me a lot about how the system works on a lower level, so I'd like to try it with some other ones.


[dns_repo]: https://github.com/pcarleton/dumb-dig
