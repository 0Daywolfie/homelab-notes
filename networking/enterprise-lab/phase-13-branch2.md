# Phase 13: Second Branch Site (Branch 2) - VERIFIED WORKING

## What was built
Added a third physical site to the topology - BR2-RTR, BR2-SW, 
PC-BR2-1, PC-BR2-2 - connected to HQ-RTR over a new WAN link.

## Ran out of serial ports on HQ-RTR
HQ-RTR's single HWIC-2T module was already fully used (Serial0/1/0 
to BR-RTR, Serial0/1/1 to ISP-RTR). Added a second HWIC-2T module 
to HQ-RTR (power off, add module, power on) - this created 
Serial0/0/0 and Serial0/0/1, giving room for the new WAN link.

## Addressing
- WAN link (HQ-RTR Serial0/0/0 <-> BR2-RTR Serial0/0/0): 
  10.0.1.0/30
- Branch 2 LAN: 192.168.51.0/24 (full /24 from the start, applying 
  the lesson from Branch 1's /30-too-small mistake)
- OSPF Area 1 (same area as Branch 1 - both branches are non-
  backbone sites)

## Configuration applied proactively (lessons from Branch 1's 
## troubleshooting applied upfront instead of discovered live)
- ip nat inside marked on HQ-RTR's new serial interface immediately, 
  not discovered as a bug afterward
- ip helper-address configured on BR2-RTR's LAN interface from the 
  start, pointing to HQ-RTR (192.168.100.2)
- Access-list 1 updated to include 192.168.51.0/24 for NAT
- DHCP pool (BRANCH2-POOL) added on HQ-RTR from the start
- No VLAN segmentation needed at Branch 2 (flat single-VLAN design, 
  same as Branch 1) - Fa0/1, Fa0/2, and the uplink port all sit in 
  default VLAN 1 by design, no explicit VLAN config required

## Verification - CONFIRMED WORKING ON FIRST ATTEMPT
- HQ-RTR shows 3 OSPF neighbors in FULL state: HQ-CORE, BR-RTR, 
  BR2-RTR
- PC-BR2-1 obtained a DHCP lease correctly on the first try
- PC-BR2-1 -> 192.168.20.1 (HQ IT, cross-site): SUCCESS, 0% loss
- PC-BR2-1 -> 8.8.4.1 (internet via NAT): SUCCESS, 0% loss

No troubleshooting marathon required this time - applying the two 
real bugs found during Branch 1's setup (VLAN mismatch, missing 
NAT inside) proactively during Branch 2's build meant it worked 
cleanly the first time.

## Key lesson
Documenting root causes thoroughly (not just the fix, but why it 
happened) pays off directly - the same class of mistakes did not 
repeat when building a near-identical second site, because the 
underlying principles (mark every NAT inside path, always size 
LAN subnets properly, always configure DHCP relay proactively) 
were internalized rather than just patched reactively.

## Next phase
- Wireless (WLC + AP) - still has an outstanding cabling question
- IoT segment
