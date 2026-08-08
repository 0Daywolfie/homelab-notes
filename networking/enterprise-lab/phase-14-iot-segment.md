# Phase 14: IoT/Voice Segment - CONFIGURED, PENDING FINAL DEVICE VERIFICATION

## What was built
Created VLAN 70 (IoT) as an isolated segment - same security pattern 
as Guest VLAN 99, but stricter: also blocks Wireless (60) and Guest 
(99) traffic, not just wired department VLANs, since IoT devices 
have no legitimate reason to reach any of those.

    VLAN 70, SVI 192.168.70.1/24, ip helper-address to HQ-RTR, 
    added to OSPF area 0

    IOT-RESTRICT ACL on HQ-CORE's Vlan70 SVI (inbound):
    deny to 192.168.10.0/24, 20.0/24, 30.0/24, 40.0/24, 60.0/24, 
    99.0/24
    permit ip any (internet allowed)

    NAT access-list updated, IOT-POOL DHCP pool added on HQ-RTR

## Device used - IP Phone (no IP camera available in this Packet 
## Tracer version)
Connected an IP Phone to HQ-ACC2 Fa0/4, assigned to VLAN 70. This 
device type doubles as a realistic example of voice VLAN 
segmentation, a genuine standard enterprise pattern distinct from 
data VLANs (QoS and security separation for voice traffic).

## Status - config verified, device lease pending
- VLAN 70 confirmed created and active via show vlan brief
- Fa0/4 confirmed correctly assigned via show interfaces status
- IP Phone's Config screen has no manual IP option - relies 
  entirely on automatic DHCP with no visible confirmation in its 
  own GUI
- Checked show ip dhcp binding on HQ-RTR - no 192.168.70.x lease 
  present yet at time of writing

This is consistent with the DHCP timing lag pattern seen 
repeatedly throughout this build (Phase 7, Phase 10, Phase 12) 
rather than a new configuration gap - the underlying pool, relay, 
and VLAN assignment are all confirmed correct. Expected to resolve 
with time/a reset, not treated as an unresolved bug.

## Lab status - BUILD COMPLETE (14 phases)
This closes out the planned two-day enterprise lab build:
- Full 3-site topology (HQ, Branch 1, Branch 2)
- VLANs, trunking, SVI inter-VLAN routing
- Single-area then multi-area OSPF with a working ABR
- NAT/PAT
- DHCP across every VLAN on every site
- Port security
- ACL-based segmentation (Guest and IoT)
- NOC with centralized syslog and NTP
- Wireless (autonomous AP with WPA2-PSK)
- IoT/Voice VLAN segmentation

Pausing active lab work here to return focus to Network Defense 
coursework toward the Junior Cybersecurity Analyst certification. 
This lab succeeded at its original purpose - reigniting motivation 
through hands-on practice after losing steam on the coursework.

## Future ideas (not started)
- Scale up Branch 2 with its own VLANs/access switches
- Export this design into GNS3/EVE-NG paired with the existing 
  Kali homelab VM for genuine offensive security practice 
  (Packet Tracer itself has no real attack tooling)
- Confirm final IoT phone DHCP lease when returning to this lab
