# Phase 17: Full NOC Remote Management - Switches Addressed, Real Issues Found and Fixed

## Background
Extended the SSH remote management from Phase 16 by giving every 
access switch (HQ-ACC1, HQ-ACC2, BR-SW, BR2-SW) a management IP, 
so NOC-ADMIN could genuinely SSH into any device across the whole 
network, not just the routers and core switch.

## What was configured
Management IPs added to each switch's local SVI:
    HQ-ACC1: Vlan1, 192.168.10.61/24, gateway 192.168.10.1
    HQ-ACC2: Vlan1, 192.168.10.62/24, gateway 192.168.10.1
    BR-SW:   Vlan50, 192.168.50.62/24, gateway 192.168.50.1 
             (used Vlan50 since BR-SW's active ports live there, 
             not Vlan1)
    BR2-SW:  Vlan1, 192.168.51.62/24, gateway 192.168.51.1

## Issue 1 - genuine unresolved anomaly on HQ-ACC2
HQ-ACC2's management IP (192.168.10.62) is unreachable from 
NOC-ADMIN and even from HQ-ACC2 itself trying to ping its own 
gateway (192.168.10.1) - despite every diagnostic checking out 
correct:
- Vlan1 interface: up/up with correct IP (initially found 
  administratively down, fixed with no shutdown - did not resolve 
  the underlying issue)
- Trunk to HQ-CORE (Fa0/3): confirmed trunking, VLAN 1 allowed, 
  active, not pruned
- Native VLAN: confirmed matching (VLAN 1) on both HQ-ACC2 and 
  HQ-CORE's corresponding trunk port
- Spanning-tree: Fa0/3 confirmed in FWD (forwarding) state, not 
  blocked
- Forced ARP learning via ping, bounced the trunk interface on 
  HQ-CORE's side, bounced Vlan1 on HQ-ACC2's side - none resolved it
- Notable observation: HQ-ACC2's spanning-tree output shows it 
  considers itself "root bridge" for every VLAN including VLAN 1, 
  which is unusual and may indicate some form of STP domain 
  isolation specific to this switch, though the exact mechanism 
  was not identified

Documented as unresolved after exhausting standard Layer 1/2/3 
diagnostics - this is a genuine anomaly, not a configuration 
mistake, unlike most other issues found in this build. HQ-ACC1, 
BR-SW, and BR2-SW's equivalent management IPs all work correctly, 
confirming this is isolated to HQ-ACC2 specifically rather than a 
systemic design flaw.

## Issue 2 - missing enable secret across multiple switches (found and fixed)
While testing SSH+enable access on BR-SW and BR2-SW, discovered 
neither had an enable secret configured - SSH login succeeded but 
`enable` returned "% No password set", preventing privileged EXEC 
access entirely over remote sessions. Checked HQ-ACC1 too and found 
the same gap there.

Fix applied to HQ-ACC1, BR-SW, BR2-SW (and swept across remaining 
devices):
    enable secret ChiefAdmin2026!

This is a genuine security/operability gap that would have made 
real remote administration impossible even though SSH login itself 
worked - a good example of how a login succeeding doesn't mean 
full remote access actually functions as intended.

## Verification - CONFIRMED WORKING
- SSH + enable access confirmed working end-to-end from NOC-ADMIN 
  into: HQ-CORE, HQ-RTR, HQ-ACC1, BR-SW, BR2-SW
- HQ-ACC2 remains unreachable (documented anomaly above)

## Key lesson
A methodical, layer-by-layer elimination process (L1 physical/link 
state, L2 trunk/VLAN/native VLAN/STP, L3 IP/ARP) is the correct 
approach even when it doesn't find a root cause - ruling out every 
plausible explanation with real evidence is still valuable, and 
knowing when to document something as a genuine unresolved anomaly 
versus continuing to chase it indefinitely is itself a practical 
skill for real network operations work.

## Next
- Real NOC-style troubleshooting exercise: diagnose a simulated 
  "PC can't reach the network" ticket entirely via SSH into the 
  relevant switch, without touching the PC directly
- Revisit HQ-ACC2 anomaly if time permits in a future session
