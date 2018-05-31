---
layout: post
title: "TIL: There are 16 T1 Internet Service Providers"
---

I've been collecting information for a "Practical Guide to Linux Networking" by reading material I can find.  Two things I've been reading recently are:

* [A Practical Guide to (Correctly) Troubleshooting with Traceroute](https://www.nanog.org/meetings/nanog47/presentations/Sunday/RAS_Traceroute_N47_Sun.pdf) 
  * a presentation by Richard A Steenbergen
* [Compute Networking: A Top Down Approach](https://www.amazon.com/Computer-Networking-Top-Down-Approach-6th/dp/0132856204) 
  * a textbook by Kurose and Ross

In the traceroute guide, Richard mentions that it's useful to find the boundaries of your network and that it's also useful to be able to translate DNS names to ISP's. This seemed highly unlikely that I would ever be able to do this given the examples shown, like:

```
p16-1-0-0.r21.asbnva01.us.bb.verio.net
ldn-bb2-link.telia.net
tbr2.wswdc.ip.att.net
xe-3-0-0.cr1.nyc3.us.nlayer.net
te2-4.ar5.PAO2.gblx.net
```

I was imagining a huge array of international cable providers like Comcast in my area that I would need to be familiar with in order to have any chance.  However, "Compute Networking", they mention that there are only 16! 

Here they are:

|Provider | Country |
|---|---|
| AT&T  |  USA |
| CenturyLink (formerly Level3, Global Crossing (gblx) and some others) | USA |
| Deutsche Telekom AG (ICSS) | Germany |
| GTT (formerly Tinet & nLayer) | USA |
| KPN International | Netherlands |
| Liberty Global | UK |
| NTT Communications (formerly Verio) | Japan |
| Orange (OpenTransit) | France |
| PCCW Global | Hong Kong |
| Sprint | Japan |
| Tata Commmunications | India |
| Telecom Italia Sparkle | Italy |
| Telxius (subsidiary Telefonica) | Spain |
| Telia Carrier | Sweden |
| Verizon Enterprise Solutions (UUNET & XO Communications) | USA |
| Zayo Group (formerly AboveNet) | USA |



I wanted to find what some of their hostnames looked like. So I tried some random tracerouting starting with the company name `.net` also trying some variations of former names.  This worked pretty well! I also tried in TCP mode with `sudo traceroute -T` which uncovered a few that eluded the first attempts.  Here are the non-exhaustive results.



### AT&T

```
$ traceroute att.net
...
 8  cr2.sffca.ip.att.net (12.122.149.134)  75.585 ms  76.402 ms  75.958 ms
...
10  cr2.dlstx.ip.att.net (12.122.2.81)  78.300 ms  77.370 ms  78.694 ms
```

We can see we jumped from SF over to Dallas Texas!



### CenturyLink

I tried `centurylink.com` but that didn't give me anything, so I tried:

```
$ traceroute level3.net
...
 6  lag-14.ear2.SanJose1.Level3.net (4.68.72.101)  18.185 ms  16.268 ms  17.309 ms
 7  * * *
 8  4.68.88.74 (4.68.88.74)  39.986 ms  41.772 ms  41.428 ms
 9  * * *
10  Level3IsNowCenturylink.com (4.68.80.110)  41.310 ms  41.700 ms  36.501 ms

# Similarly for globalcrossing.com
10  thenewcenturylink.com (4.68.80.110)  43.422 ms  37.603 ms  37.615 ms
```



### Deutsche Telekom AG (ICSS)

```
$ sudo traceroute -T telekom.com
...
9  m-eb7-i.M.DE.NET.DTAG.DE (217.5.69.14)  180.473 ms  175.974 ms  176.948 ms

```



### GTT Communications

```
$ traceroute tinet.net
...
 8  xe-7-3-0.cr0-trn3.ip4.gtt.net (89.149.184.162)  180.859 ms  179.917 ms  181.247 ms
 9  it-farm-gw2.ip4.gtt.net (77.67.94.202)  203.231 ms  198.183 ms  203.110 ms

```



### KPN International

```
$ traceroute kpn.com 
 6  lag-14.ear2.SanJose1.Level3.net (4.68.72.101)  25.916 ms  17.694 ms  18.140 ms
 7  ae-237-3613.edge6.Amsterdam1.Level3.net (4.69.162.242)  158.544 ms  151.896 ms  153.006 ms
 8  IPTRIPLEPLA.edge6.Amsterdam1.Level3.net (212.72.47.250)  157.888 ms  159.041 ms  158.746 ms
 9  * * *
10  cca-iaas-cr01.net.kpnvdc.nl (145.128.23.193)  159.382 ms  159.334 ms  159.067 ms
11  145.128.23.242 (145.128.23.242)  159.278 ms  159.481 ms  159.424 ms
12  apd-iaas-cr01.net.kpnvdc.nl (145.128.13.146)  171.001 ms  165.615 ms  165.130 ms
13  * * *
14  static.kpnvdc.nl (145.128.64.11)  160.604 ms  161.201 ms  161.399 ms

```

### Sprint

```
$ sudo traceroute -T sprint.net
...
18  sl-sprin881-320471-0.sprintlink.net (144.223.33.34)  85.353 ms  84.287 ms  97.713 ms
```



### Open Questions

After digging in to this stuff, I still had some questions.

* What is an AS number? And how does it translate to IP CIDR ranges?
* Is there a map of T1 endpoints somewhere?
* How much does someone pay to send traffic over a T1?

### Other resources:

* [Wikipedia: Tier 1 Network](https://en.wikipedia.org/wiki/Tier_1_network)
* `mtr`is like traceroute but continuous (I wanted easy copy-pasteable output, so I stuck with traceroute.)
* `traceroute bad.horse` (Just try it)
  * Also `openssl s_client -connect signed.bad.horse:443 -servername signed.bad.horse < /dev/null`
