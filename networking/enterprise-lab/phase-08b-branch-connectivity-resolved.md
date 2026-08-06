# Phase 8 Follow-up: Branch PC Connectivity - ROOT CAUSE FOUND AND FIXED

## Background
After the Phase 8 subnet redesign (192.168.101.0/30 to 
192.168.50.0/24), PC-BR1 and PC-BR2 could not ping their gateway, 
HQ, or the internet. Initially suspected a Packet Tracer simulation 
quirk (matching the pattern from Phase 7's DHCP/ACL display issues) 
and moved on to other work. Returned to it after completing 
Phase 9 (multi-area OSPF) and diagnosed it properly with fresh eyes.

## Root cause 1 - VLAN mismatch on the switch uplink port
Checked show interfaces status on BR-SW and found Fa0/24 (the port 
connecting to BR-RTR) was still sitting in default VLAN 1, while 
PC-BR1/PC-BR2 were correctly in VLAN 50. Confirmed via show cdp 
neighbors on BR-RTR that Layer 2 discovery could see BR-SW fine - 
the physical link worked, but the uplink port was in the wrong 
VLAN, meaning PC ARP broadcasts never reached BR-RTR's interface, 
regardless of correct IP addressing on both ends.

Fix:
    BR-SW#configure terminal
    BR-SW(config)#interface fastEthernet 0/24
    BR-SW(config-if)#switchport mode access
    BR-SW(config-if)#switchport access vlan 50
    BR-SW(config-if)#exit

Verified with show interfaces status - Fa0/24 now shows VLAN 50. 
Retested: gateway ping (192.168.50.1) and cross-site ping to HQ 
(192.168.20.1) both succeeded immediately. Internet (8.8.4.1) 
still failed.

## Root cause 2 - NAT inside designation missing on the WAN interface
Internet ping still failed with "Destination host unreachable" 
from the gateway. Checked show ip nat statistics on HQ-RTR: 
"Hits: 0 Misses: 21" - confirmed NAT was seeing the traffic attempt 
but rejecting every one of them.

Realized ip nat inside was only configured on GigabitEthernet0/0 
(facing HQ-CORE) - but Branch traffic arrives at HQ-RTR via 
Serial0/1/0 (the WAN link to BR-RTR), which had never been marked 
as an inside interface. NAT only translates traffic entering 
through interfaces explicitly marked ip nat inside - having the 
source subnet correctly listed in the access-list is not enough 
on its own if the arrival interface isn't also marked inside.

Fix:
    HQ-RTR#configure terminal
    HQ-RTR(config)#interface serial 0/1/0
    HQ-RTR(config-if)#ip nat inside
    HQ-RTR(config-if)#exit

Verified with show ip nat statistics - Inside Interfaces now lists 
both GigabitEthernet0/0 and Serial0/1/0.

## Final Verification - ALL THREE TESTS PASS
- PC-BR1 -> 192.168.50.1 (local gateway): SUCCESS, 0% loss
- PC-BR1 -> 192.168.20.1 (HQ IT, cross-site via multi-area OSPF): 
  SUCCESS, 0% loss
- PC-BR1 -> 8.8.4.1 (internet via NAT): SUCCESS, 0% loss

## Key lessons
1. When a device has correct IP addressing but nothing works at 
   all, check Layer 2 first (VLAN membership on every hop) before 
   assuming a Layer 3 problem - this issue had nothing to do with 
   IP addressing, routing, or NAT config on its own.
2. NAT's ip nat inside must be applied to every interface that 
   inside traffic could arrive through, not just the "main" 
   internal-facing interface. A router with multiple internal 
   paths (like an ABR connecting to both a LAN and a remote site 
   over WAN) needs inside marked on all of them.
3. Systematic layer-by-layer elimination (L1 physical, L2 VLAN, 
   L3 IP/routing, NAT) found two genuinely separate real bugs 
   stacked on top of each other - fixing the first one revealed 
   the second, which had been masked entirely by the first failure.
4. Don't assume "simulator quirk" too early - this issue looked 
   identical in symptom to the Phase 7 DHCP quirk (everything 
   times out, nothing obviously wrong) but was actually two real, 
   fixable configuration errors.

## Lab status update
Multi-area OSPF (Phase 9) and this Branch connectivity fix mean 
the entire two-site topology is now fully functional end-to-end: 
local traffic, cross-site traffic, and internet-bound traffic all 
verified working from both HQ and Branch.
