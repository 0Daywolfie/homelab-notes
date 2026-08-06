# Phase 9: Multi-Area OSPF - VERIFIED WORKING

## What was configured
Split single-area OSPF (Area 0 only) into two areas:
- Area 0: HQ-CORE's networks, HQ-RTR's Gig0/0 (to HQ-CORE) and 
  Serial0/1/1 (to ISP-RTR)
- Area 1: Branch - HQ-RTR's Serial0/1/0 (to BR-RTR), BR-RTR's 
  entire OSPF participation (WAN link + 192.168.50.0/24 LAN)

HQ-RTR became an ABR (Area Border Router), sitting on the boundary 
between the two areas.

On HQ-RTR:
    router ospf 1
    no network 10.0.0.0 0.0.0.3 area 0
    network 10.0.0.0 0.0.0.3 area 1

On BR-RTR:
    router ospf 1
    no network 10.0.0.0 0.0.0.3 area 0
    network 10.0.0.0 0.0.0.3 area 1
    no network 192.168.50.0 0.0.0.255 area 0
    network 192.168.50.0 0.0.0.255 area 1

## Issue encountered - area mismatch (expected OSPF behavior)
After updating HQ-RTR's side but before updating BR-RTR's side, 
hit a real OSPF error:

    %OSPF-4-ERRRCV: Received invalid packet: mismatch area ID, 
    from backbone area must be virtual-link but not found 
    from 10.0.0.1, Serial0/1/0

This is not a bug - OSPF requires both ends of a link to agree on 
the area ID before forming a neighbor adjacency. The neighbor 
relationship disappeared entirely from `show ip ospf neighbor` 
during the mismatch window. Resolved once BR-RTR's config was 
updated to match Area 1 - neighbor relationship reformed to FULL 
state automatically.

## Verification - CONFIRMED WORKING
`show ip ospf interface brief` on HQ-RTR confirms ABR status - 
interfaces genuinely split across two areas:
- Gig0/0 (192.168.100.2) - Area 0
- Se0/1/1 (203.0.113.1) - Area 0  
- Se0/1/0 (10.0.0.1) - Area 1

From HQ-CORE (pure Area 0 router), Branch's routes now correctly 
show as O IA (inter-area) instead of plain O (intra-area):
    O IA 192.168.50.0/24 [110/66] via 192.168.100.2
    O IA 10.0.0.0/30 [110/65] via 192.168.100.2

This confirms the area boundary is genuinely functioning - routes 
originating outside a router's own area are correctly tagged as 
inter-area when learned through an ABR.

## Key concept confirmed
An ABR (Area Border Router) sits in multiple OSPF areas 
simultaneously and is responsible for summarizing/advertising 
routes between them. From the ABR's own perspective, routes in 
directly-attached areas show as regular O (intra-area) regardless 
of which area they're in - O IA only appears on routers that 
receive a route which crossed an area boundary via an ABR, not on 
the ABR itself. This directly relates to concepts tested in a 
recent AI Trainer interview (ABR/ASBR questions).

## Next phase
- Resolve outstanding PC-BR1/PC-BR2 connectivity issue (flagged 
  in phase-08)
- Second branch site
- NOC - centralized monitoring
- Wireless - WLC + AP
- IoT segment
