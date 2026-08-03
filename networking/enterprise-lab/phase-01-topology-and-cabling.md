# Phase 1: Topology Build and Cabling

## Devices placed
- HQ-CORE (3560-24PS) - Layer 3 core switch
- HQ-ACC1, HQ-ACC2 (2960) - access switches
- BR-SW (2960) - branch access switch
- HQ-RTR, BR-RTR, ISP-RTR (1941) - routers
- HQ-DHCP-DNS, HQ-WEB, Public-WEB - servers
- 8 PCs across HQ (IT, Sales, Finance) and Branch

## Cabling completed
- 8 PC-to-switch links (Copper Straight-Through)
- HQ-ACC1/HQ-ACC2 to HQ-CORE
- HQ-CORE to HQ-RTR, HQ-DHCP-DNS, HQ-WEB
- BR-SW to BR-RTR
- ISP-RTR to Public-WEB
- HQ-RTR to BR-RTR (Serial DCE)
- HQ-RTR to ISP-RTR (Serial DCE)

## Issues encountered and fixed
- Router serial ports required adding HWIC-2T module (1941 has no 
  built-in serial ports)
- Interface naming assumptions were wrong (Serial0/1/0 not Serial0/0/0) - 
  always verify with `show ip interface brief` rather than assuming
- All router interfaces default to administratively down - required 
  manual `no shutdown` on every WAN and uplink interface
- Cable initially miswired between BR-RTR and ISP-RTR directly - 
  corrected to route both WAN links through HQ-RTR as the central hub
- Public-WEB was cabled to Gi0/0 on ISP-RTR, but Gi0/1 was enabled 
  by mistake first - verified actual cable port before fixing
- Set clock rate 64000 on HQ-RTR's serial interfaces (DCE side)

## Verification
- `show ip interface brief` confirmed all intended interfaces show 
  up/up on HQ-RTR, BR-RTR, and ISP-RTR

## Next phase
- Configure trunk ports on HQ-CORE
- Create and assign VLANs
