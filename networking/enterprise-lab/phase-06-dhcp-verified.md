# Phase 6: DHCP Configuration - VERIFIED WORKING

## What was configured
- Assigned static IPs to HQ-DHCP-DNS (192.168.10.10) and HQ-WEB 
  (192.168.10.20) servers - both had never been addressed before, 
  had to also assign their switch ports (Fa0/3, Fa0/5) to VLAN 10 
  first since they were still sitting in default VLAN 1
- Configured 4 DHCP pools on HQ-RTR (CLI-based, not the server GUI):
  - IT-POOL: 192.168.20.0/24
  - SALES-POOL: 192.168.30.0/24
  - FINANCE-POOL: 192.168.40.0/24
  - GUEST-POOL: 192.168.99.0/24
- Excluded gateway and static server addresses from each pool to 
  prevent conflicts

## Issue encountered - DHCP requests not reaching HQ-RTR
- PCs switched to DHCP mode got 0.0.0.0 (failed lease) initially
- Root cause: DHCP requests are broadcasts and dont cross Layer 3 
  boundaries by default - HQ-CORE (where the PCs VLANs actually 
  live via SVI) had no way to forward those broadcasts to HQ-RTR 
  (where the DHCP pools are configured)
- Fixed with ip helper-address 192.168.100.2 on each VLANs SVI 
  on HQ-CORE - forwards DHCP broadcasts directly to HQ-RTR instead 
  of dropping them locally
- Verified the helper-address config was correctly applied via 
  show running-config (the shorthand show run interface vlan X 
  is not valid syntax on this IOS version)
- Some PCs still didnt pick up a lease on the first attempt even 
  after helper-address was confirmed correct - resolved by manually 
  toggling each PC from Static back to DHCP to force a fresh 
  discovery request (Packet Tracer timing quirk, not a config issue)

## Verification - DHCP CONFIRMED WORKING
show ip dhcp binding on HQ-RTR shows active leases across all 
three tested VLANs:
- 192.168.20.5, 192.168.20.6 (IT)
- 192.168.30.2, 192.168.30.3 (Sales)
- 192.168.40.2, 192.168.40.3 (Finance)

## Key concept confirmed
DHCP requests are Layer 2 broadcasts and cannot cross VLAN/subnet 
boundaries without explicit help. ip helper-address (DHCP relay) 
is the standard mechanism for centralizing DHCP services rather 
than running a separate DHCP server per VLAN/subnet.

## Next phase
- Configure ACLs for Guest VLAN restriction (internet-only access, 
  blocked from internal department resources)
