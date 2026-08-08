# Enterprise Network Lab

A 3-site enterprise network built and configured entirely in Cisco 
Packet Tracer, designed to practice and demonstrate real CCNA and 
network engineering concepts end-to-end - not just individual 
commands, but a full working topology with genuine troubleshooting 
along the way.

## Topology overview

- **HQ** - Layer 3 core switch (HQ-CORE), two access switches, 
  6 VLANs, dual internal servers, NOC monitoring, wireless AP
- **Branch 1** - single-VLAN branch site connected over a WAN link
- **Branch 2** - second branch site, added later to prove the 
  design scales
- **Internet edge** - simulated ISP router and public web server 
  for realistic NAT testing

All sites route through HQ as the central hub - branches reach the 
internet via HQ's NAT, not their own direct connections 
(centralized internet breakout, a common real-world enterprise 
pattern).

## What's implemented

| Area | Details |
|---|---|
| VLANs & Trunking | 7 VLANs (Management, IT, Sales, Finance, Wireless, Guest, IoT), 802.1Q trunking |
| Inter-VLAN Routing | SVI-based routing on a Layer 3 core switch |
| Dynamic Routing | OSPF, single-area then upgraded to multi-area with a working ABR |
| NAT/PAT | Full internal network sharing one public IP via PAT |
| DHCP | Every VLAN across every site uses dynamic addressing |
| Security | Port security (sticky MAC, violation shutdown), ACL-based VLAN isolation for Guest and IoT |
| Monitoring | Centralized syslog + NTP (NOC) |
| Wireless | Autonomous AP with WPA2-PSK, dedicated wireless VLAN |
| IoT/Voice | Isolated VLAN for IoT/voice devices, stricter ACL than Guest |

## Phase-by-phase documentation

Each phase file documents what was configured, real issues hit 
along the way, how they were diagnosed and fixed, and verification 
evidence (actual command output, not just "it works"):

1. Topology and cabling
2. VLANs, trunking, access port assignments
3. SVI inter-VLAN routing
4. OSPF (single-area)
5. NAT/PAT
6. DHCP
7. ACL - Guest VLAN restriction
8. Port security + branch subnet redesign (a real design flaw 
   caught and fixed)
8b. Branch connectivity root-cause writeup (VLAN mismatch + NAT 
    scope - two real, distinct bugs found through systematic 
    elimination)
9. Multi-area OSPF (ABR/ASBR)
10. Branch DHCP
11. NOC monitoring (syslog + NTP)
12. Wireless (AP + WLC)
13. Second branch site
14. IoT/Voice segment

## Why this exists

Built as a hands-on project to stay motivated while working toward 
a Junior Cybersecurity Analyst certification - proof that theory 
and practice reinforce each other. Every phase here reflects real 
troubleshooting, including the mistakes, not just the clean final 
config.

## Possible future work

- Scale Branch 2 with its own VLANs and access infrastructure
- Migrate this design into GNS3/EVE-NG for genuine offensive 
  security practice against a self-built network
