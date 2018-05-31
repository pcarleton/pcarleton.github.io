---
layout: post
title: "Where did my packets go?"
---

Packet Loss Demystified

Outline: 

- Motivate
-- Packet loss: send too many packets, some get dropped
-- How many is too many?
-- How do I know if my server is dropping packets?
-- Where along the chain do they get dropped?
-- Which packets get dropped?



Model of a network
Client -> Send Buffer -> Wire -> Receive Buffer -> Server

Router
Wire -> Receive Buffer -> Routing Logic -> Send Buffer -> Wire


-- Why Buffers? If not, then if server stopped listening for a second, it would miss information.


Wait a second, what's putting stuff in the buffer then?

NIC card, poll driven or interrupt driven.


* How much buffer in a broadcom chip?
-- My laptop:  Broadcom Corporation BCM43224 (via `lspci`)
-- Tried docs page: https://www.broadcom.com/products/wireless/wireless-lan-infrastructure/bcm43224#documentation
-- https://stackoverflow.com/questions/17154759/how-much-memory-does-a-common-nic-have
-- Not a given that they have their own memory, may use host memory
-- https://www.ibm.com/support/knowledgecenter/en/SSQPD3_2.6.0/com.ibm.wllm.doc/nicringbuffers.html
-- ethtool -g eth0 - shows current NIC buffer sizes
-- ethtool -i eth0 - shows current NIC driver


https://github.com/torvalds/linux/blob/master/drivers/net/wireless/broadcom/brcm80211/brcmsmac/main.h#L185:27

Looks like 6 packets of TX/RX buffer


https://people.cs.clemson.edu/~westall/853/tcpperf.pdf

NIC -> DMA (Direct Memory Access) -> Ring Buffer -> "SoftIrq"?? -> Socket RECV -> Application

http://www.linuxjournal.com/content/queueing-linux-network-stack
Good queue diagram

ip & ifconfig can show txqueuelen. this is the default queue len for certain qdiscs.

(I guess the dropped list in those metrics are from the qdisc queue?)


iptables drops from what queue?

https://www.digitalocean.com/community/tutorials/a-deep-dive-into-iptables-and-netfilter-architecture

netfilter provides hooks into networking stack (not sure if pre-or-post qdisc)

iptables is stateful, packets are routed in relation to previous packets (what does that mean in practice?)

"RAW" lets you opt a packet out of connection tracking. -- Not sure why you would want this but it sounds like a way to get around the conntrack limit in a bind.

How do I see the state of current tracked connections? `conntrack -L` (after `sudo apt-get install conntrack`)


https://unix.stackexchange.com/questions/288959/how-is-the-ifb-device-positioned-in-the-packet-flow-of-the-linux-kernel
https://upload.wikimedia.org/wikipedia/commons/3/37/Netfilter-packet-flow.svg
Great diagram of iptables route application.  Shows that iptables effects things that happen after the qdisc happens


Where does TCP dump plug in to get its data?
http://linux-ip.net/gl/tc-filters/tc-filters-node3.html
^ That page indicates that it captures before the ingress qdisc which is kind of surprising.

https://stackoverflow.com/questions/3628330/in-tools-like-tcpdump-when-exactly-are-the-network-packets-captured
https://wiki.linuxfoundation.org/networking/kernel_flow
^ TCPdump grabs on transmit during the "dev_hard_start_xmit()" for tx, this is after qdisc stuff happens
rx?? -- similarly before the ingress qdisc.



OPEN QUESTIONS: 
* How do I see when a socket is dropping packets?
* How do I see when a qdisc is dropping packets?
* Can I see when a NIC is dropping packets?



https://serverfault.com/questions/533513/how-to-get-tx-rx-bytes-without-ifconfig
I guess this shows ring buffer dropped bytes, i.e. after the NIC?
http://www.onlamp.com/pub/a/linux/2000/11/16/LinuxAdmin.html
Dropped by the device driver -- is that before qdisc?


Tidbits

* traffic control
** simulate packet loss
**  qdisc "queuing discipline"
** pfifo_fast vs. fair_codel
** ring buffer
* Dropped when write buffer circles around to read buffer?
** FIFO - drop new when queue is full (tail drop)
* Are new packets dropped? or old packets? (maybe fair_codel handles that)




[ring-buffer]: https://en.wikipedia.org/wiki/Circular_buffer
[packet-loss]: https://en.wikipedia.org/wiki/Packet_loss
[tcp-congestion-control]: https://en.wikipedia.org/wiki/TCP_congestion_control

[network-interface-controller]:https://en.wikipedia.org/wiki/Network_interface_controller



Tidbits:

Bufferbloat
-  TCP uses dropped packets to reduce throughput bc it assumes its a congested link.
- How does TCP know the packet is dropped?
- - Must be either a timeout, or receiving an out of order response?
- - duplicate ACK , or retransmission timeout (RTO)
- - duplicate ACK happens when the receiver gets message 5 before message 2, but it sends sequence number 2 back because its missing an earlier packet.  This indicates a retrasmission should happen. (usually set to 4 total ACKs with the same number)


- Active Queue Management (AQM)
- - Proactively drop packets when nearing congestion to avoid full buffers/ bufferbloat, and global synchronization of TCP (e.g. when they hit congestion, then slowly ramp up together)

- erlang unit

- southbridge?
