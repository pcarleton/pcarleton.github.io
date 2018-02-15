---
layout: post
title: "Meandering History of DNS Authority"
---

As a follow up to my DNS post, I started trying to answer "Who's in charge of all this?".

Basically, I wanted to know what keeps me from claiming a different domain name or for that matter for claiming a different IP address.

I went down a rabbit hole of RFC's and never quite pulled out the answer to my question, but I wanted to post what I did find out since it's been languishing in my drafts for too long.


## Summary

Systems were each keeping track of the list of "names" of hosts and their "addresses" (which seem to just be decimal numbers, it's unclear what protocol these addresses were for as this was before the IP protocol was developed.).

In 1973, the NIC put out an official list of host names and addresses, afterwhich somebody suggested that the NIC maintain a file that other hosts can pull down.  The NIC agreed, and started serving a "HOSTS.TXT" file in ASCII format with the host name and addresses available via FTP.

In 1981, Internet Protocol (IP) was introduced, and J. Postel put out a list of IP ranges assigned to existing networks.  These networks were free to assign IP addresses in that range as they saw fit.

In 1982, Feinler updates the HOSTS.TXT format to use IP addresses and to specify networks, gateways and hosts separately.




### 1969

First computers linked

Engelbart volunteers Stanford Research Institute (SRI) to be the Network Information Center (NIC) since he has NLS (oN-Line System) which could help manage it.



### 1972

Jake Feinler transfers into ARC (Augmentation Research Center) to work on NIC.

[Socket catalog request - RFC 322](https://tools.ietf.org/html/rfc322)

[RFC 349 - Proposed Socket Numbers](https://tools.ietf.org/html/rfc349)
  Postel suggests there be a czar for socket numbers for standard protocols
  and that it be him

[Socket Number List - RFC 433](https://tools.ietf.org/html/rfc433)
  Postel specifies the socket number assignent

### 1973  

[RFC 597 - List of hosts on ARPANET](https://tools.ietf.org/html/rfc597)
	interestingly the address numbers are between 0 and 255
	
	OFFICE-1 the NIC host is on a PDP-10 along with NLS
	Also offers "TENEX"

[RFC 606 - Host Names On-Line](https://tools.ietf.org/html/rfc606)
	L. Peter Deutsch recommends a centralized list of host names

### 1974

[RFC 608 - Host Names On-Line](https://tools.ietf.org/html/rfc608)
	Jake commits to updating a text file
	File lives at <NETINFO>HOSTS.TXT
	Host "OFFICE-1" Host Address = 43 decimal

	^^ so Host Addresses were just numbers

### 1977
NLS sold to Tymshare, with it the machine NIC ran on, so NIC switches to DEC-10

[RFC 739 - Assigned Numbers](https://tools.ietf.org/html/rfc739)
  Postel lists the assigned "link" numbers and socket numbers for protocols.
  Not clear to me exactly what a "link" is.  I'm not sure how packets were
  routed, this was pre-IP though, so maybe it was in a really weird way

  Refers to the "internetwork protocols"

1980

[RFC 760 - Internet Protocol](https://tools.ietf.org/html/rfc760)
  

### 1981
Convert from NCP to TCP/IP (NCP was original Arpanet protocol)
TCP/IP created?

DDN created

[RFC 790 - Assigned Numbers](https://tools.ietf.org/html/rfc790)
  Class A,B,C networks described.
  IP ranges assigned to different networks

[RFC 796 - Address Mappings](https://tools.ietf.org/html/rfc796)
  Class A,B,C networks, describes how network specific addresses correspond
  to IP addresses.

[RFC 791 - Internet Protocol (IP)](https://tools.ietf.org/html/rfc791)
  This is an update


### 1982
[RFC 810 - Host Table Specification](https://tools.ietf.org/html/rfc810)
  > It can be obtained by connecting to host SRI-NIC (10.0.0.73) from your local FTP server, logging in as user=ANONYMOUS, password=GUEST, and doing a 'get' on <NETINFO>HOSTS.TXT.
	
		

[RFC 811 - Hostname Server](https://tools.ietf.org/html/rfc811)
  SRI-NIC is mentioned to be running on a Foonly
  Describes command interface for looking up the host addresses for machines by name

### 1983 
Deadline for military to adopt TCP/IP


### 1984 

Domain Name requirements 
[RFC 920](http://www.rfc-editor.org/rfc/rfc920.txt)
> The administrator of a level N domain must register with the
registrar (or responsible person) of the level N-1 domain.  This
upper level authority must be satisfied that the requirements are
met before authorization for the domain is granted.

Lays out the information necessary to register a domain.  Essentially:
email hostmaster@sri-nic.arpa with your information including the IP addresses of the name servers that will host the DNS information.

The IP address listed is of the typical x.x.x.x format.

### 1985
[RFC 995: Hostname server](https://tools.ietf.org/html/rfc953)
(Feinler update to RFC 811)


### 1987
NIC took over maintaining assigned numbers and become Arpanet/DDN's naming auhtority

[RFC 1035 - DNS protocol specification](https://tools.ietf.org/html/rfc1035)


### 1991 
NIC contract awarded to Network Solutions Inc (NSI).

### 1993
NSI adds root server

### 1994

RFC 1591 (Postel):
internic.net - registers second level domains under the TLD's (com, edu, org, gov, net)

now is US Department of Commerce


.mil - DDN 
.int - PVM at ISI.edu (USC)


### 1998

ICANN established



## Unanswered Questions
* If NIC did the host names, what did IANA do?
* What is TIP? and what is a TIP phone number?
* What was Namedroppers?
* What format were addresses before IP? How did things get routed?



## Further Reading:

[History by Jake Feinler](http://ieeexplore.ieee.org/document/5551028/?reload=true)
(a)
[Verisign a root server](http://a.root-servers.org/)
pre-1991 Network Information Center (NIC) at Stanford Research Institute (SRI)
SRI operated one of 3 root name servers.

(b) USC started in 1987
[USC b root server](http://b.root-servers.org/)

(c) also started in 1987
[Cogent c root server](http://c.root-servers.org/)

(d) 1988
[University of Mayland d root server](http://d.root-servers.org/)


[IANA Wikipedia](https://en.wikipedia.org/wiki/Internet_Assigned_Numbers_Authority)

[IANA's role - RFC 1591](https://tools.ietf.org/html/rfc1591)
"It is extremely unlikely that any other TLDs will be created." (lol)



