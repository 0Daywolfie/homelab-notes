# Phase 18: VLAN 1 Management Reachability - Unresolved Anomaly (Documented, Not Chased Further)

## Background
Continued Phase 17's work by attempting a realistic NOC troubleshooting 
exercise (diagnose "PC-SALES2 can't reach internet" purely via SSH 
into HQ-ACC1). Discovered HQ-ACC1's own management IP had stopped 
being reachable, despite working earlier in the same session.

## Real issue found and fixed - HQ-ACC1's trunk config had disappeared
Checked HQ-ACC1's full running-config and found Fa0/5 (its uplink 
trunk to HQ-CORE) had reverted to a blank, non-trunking interface - 
no `switchport mode trunk` at all, despite this having been 
configured since Phase 2 and working throughout the entire build. 
Root cause not identified with certainty - suspected an accidental 
interface reset or overwrite during the heavy config work in 
Phase 17 (enable-secret sweep across multiple devices).

Fixed:
    interface FastEthernet0/5
    switchport mode trunk

Confirmed restored via show interfaces trunk - Fa0/5 correctly 
showing 802.1q trunking, VLAN 1 allowed and active again.

## Anomaly - VLAN 1 management reachability still failed after the fix
Despite the trunk being genuinely restored and verified correct, 
ping and SSH to HQ-ACC1's management IP (192.168.10.61) continued 
to fail completely. The exact same symptom pattern as HQ-ACC2's 
earlier unresolved issue in Phase 17 - both switches use VLAN 1 
for their management SVI, both became unreachable despite correct-
looking configuration.

## Diagnostics performed (all came back "correct" despite the failure)
- Trunk status, VLAN 1 allowance, not pruned: confirmed correct
- Native VLAN match between HQ-CORE and HQ-ACC1: confirmed matching
- Spanning-tree state for VLAN 1 on both switches: confirmed FWD 
  (forwarding), not blocked - though notably, HQ-CORE's root port 
  selection for VLAN 1 pointed toward HQ-ACC2's link (Fa0/2) rather 
  than HQ-ACC1's (Fa0/1), an unusual but not necessarily incorrect 
  STP outcome
- ARP table on HQ-CORE: no entry ever appeared for either 
  192.168.10.61 or .62, despite forced ping attempts and interface 
  bounces
- Tested giving HQ-CORE's own Vlan1 SVI an IP address (theory: 
  transit device might need L3 presence on a VLAN to properly 
  forward it) - did not resolve the issue

## Pattern identified
Both switches affected by this anomaly (HQ-ACC1 and HQ-ACC2) share 
one thing in common: their management SVI is on VLAN 1. Every other 
management IP in this build, configured on non-default VLANs 
(HQ-CORE's various department SVIs, BR-SW's Vlan50-based management 
IP), has worked without issue throughout the entire project. This 
strongly suggests a VLAN-1-specific Packet Tracer simulation quirk, 
rather than a general trunking or routing problem.

## Decision - stopped chasing, documented instead
After exhausting Layer 1 (physical/link), Layer 2 (trunk, VLAN, 
native VLAN, spanning-tree), and Layer 3 (SVI, ARP, routing) 
diagnostics with no resolution, made the deliberate call to stop 
rather than continue indefinitely. This mirrors the same disciplined 
approach used for the Phase 7 DHCP/ACL display anomaly earlier in 
the build - real, methodical elimination followed by an honest 
"unresolved" conclusion rather than an artificially forced fix.

## Key lesson - a genuine best practice discovered through failure
Avoiding VLAN 1 for management purposes is actual real-world Cisco 
best practice, independent of this anomaly - VLAN 1 is the default/
well-known VLAN on every Cisco device out of the box, making it a 
more predictable target in real attacks, and many enterprise 
designs deliberately move all management traffic to a dedicated, 
non-default VLAN. This session's struggles inadvertently reinforced 
that lesson through direct experience rather than by having 
designed it in from the start.

## Lab status
Pausing this specific build here. Total: 18 phases, a complete 
3-site enterprise network with VLANs, multi-area OSPF, NAT, DHCP, 
port security, ACL segmentation, NOC monitoring, wireless, IoT/voice 
segmentation, and network-wide SSH remote management (with two 
switches - HQ-ACC1 and HQ-ACC2 - exhibiting the VLAN 1 management 
anomaly documented above).

## Planned next build (separate project)
A new lab, designed from scratch applying every lesson learned here:
- Dedicated non-default VLAN for all management IPs from the start 
  (not VLAN 1)
- FTP and additional protocols beyond what this build covered
- Full NOC-style administration built in as a first-class design 
  goal, not retrofitted
- Real, deliberate protocol diversity (SSH, FTP, and others) as an 
  explicit learning objective
