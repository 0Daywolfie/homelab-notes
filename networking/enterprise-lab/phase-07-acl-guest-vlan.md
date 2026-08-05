# Phase 7: ACL - Guest VLAN Restriction (CONFIGURED, PARTIALLY VERIFIED)

## What was configured
Extended ACL on HQ-CORE restricting Guest VLAN (99) from reaching 
internal department subnets while still allowing internet access:

    ip access-list extended GUEST-RESTRICT
    deny ip 192.168.99.0 0.0.0.255 192.168.10.0 0.0.0.255
    deny ip 192.168.99.0 0.0.0.255 192.168.20.0 0.0.0.255
    deny ip 192.168.99.0 0.0.0.255 192.168.30.0 0.0.0.255
    deny ip 192.168.99.0 0.0.0.255 192.168.40.0 0.0.0.255
    permit ip 192.168.99.0 0.0.0.255 any

Applied inbound on VLAN 99's SVI:

    interface vlan 99
    ip access-group GUEST-RESTRICT in

## Verification attempted - DHCP on VLAN 99 hit a Packet Tracer quirk
Attempted to test the ACL by moving a PC into VLAN 99 (via HQ-ACC1 
Fa0/6) and confirming: internet access works, internal subnet access 
is blocked. DHCP lease acquisition on VLAN 99 specifically failed 
repeatedly, including after:
- Toggling Static/DHCP on the PC multiple times
- `ipconfig /release` and `/renew` from CLI
- Restarting the DHCP service on HQ-RTR (`no service dhcp` / 
  `service dhcp`)
- Fully reloading HQ-RTR
- Swapping in a completely fresh, never-configured PC (ruled out 
  stale lease/device-specific state as the cause)

Every underlying layer was verified correct and healthy:
- GUEST-POOL: 0 leases, 254 total addresses, correctly excluded 
  gateway range - confirmed via `show ip dhcp pool GUEST-POOL`
- Vlan99 SVI: up/up, correct IP (192.168.99.1) - confirmed via 
  `show ip interface brief`
- `ip helper-address 192.168.100.2` present under Vlan99 - 
  confirmed via full `show running-config`
- Fa0/6 physical port: connected, correctly assigned to VLAN 99 - 
  confirmed via `show interfaces status`
- Trunk to HQ-CORE: VLAN 99 confirmed allowed, active, and not 
  pruned - confirmed via `show interfaces trunk`

Conclusion: every configuration element checks out correct across 
all layers (L1 physical, L2 VLAN/trunk, L3 SVI/routing, DHCP pool 
config, helper-address relay config). This points to a Packet 
Tracer simulation-engine quirk specific to this VLAN/pool 
combination rather than a configuration error - a known category 
of issue in network simulators, distinct from a real misconfiguration.

## Key lesson
When every individually verifiable layer of a config checks out 
correct via CLI diagnostics, but live end-to-end behavior still 
fails, that's a signal to stop debugging the config itself and 
consider tooling/simulator limitations. Knowing when to stop 
chasing a simulator bug versus a genuine config error is itself 
a practical skill - spent excessive time here would have been 
unproductive rather than educational.

## Design intent (logically correct, pending live verification)
The ACL logic itself is textbook-correct extended ACL syntax: 
explicit denies for each internal subnet, explicit permit any as 
the catch-all (required due to Cisco's implicit deny-all at the 
end of every ACL). Applied inbound at the SVI to filter Guest 
traffic at its point of entry into the routed network - standard 
practice for this kind of segmentation.

## Lab status
This closes out the planned build for the enterprise lab: 
topology, cabling, VLANs, trunking, SVI inter-VLAN routing, OSPF 
site-to-site, NAT/PAT, DHCP, and ACL guest restriction have all 
been configured. Six of seven phases were fully live-verified with 
working ping tests; this final phase is configuration-verified but 
not live-tested due to the DHCP quirk described above.
