# Phase 5: NAT/PAT Configuration - VERIFIED WORKING

## What was configured
- Designated HQ-RTR Gig0/0 as `ip nat inside` (facing HQ-CORE/internal network)
- Designated HQ-RTR Serial0/1/1 as `ip nat outside` (facing ISP-RTR)
- Created access-list 1 permitting all internal subnets to be NAT-translated:
  - 192.168.10.0/24, 192.168.20.0/24, 192.168.30.0/24, 192.168.40.0/24, 
    192.168.99.0/24, 192.168.101.0/30 (Branch)
- Configured PAT (NAT overload): 
  `ip nat inside source list 1 interface serial 0/1/1 overload`

## Prerequisite issue discovered and fixed - missing default route
- Initial ping test failed with "Destination host unreachable" from 
  HQ-CORE - OSPF only advertises internal networks, never advertises 
  a route to the simulated internet (ISP-RTR's network) by design
- Fixed with a static default route on HQ-RTR:
  `ip route 0.0.0.0 0.0.0.0 serial 0/1/1`
- Propagated that default route into OSPF so HQ-CORE and BR-RTR 
  would learn it too: `default-information originate` under 
  `router ospf 1`
- Verified with `show ip route` on HQ-CORE - confirmed O*E2 0.0.0.0/0 
  route learned via OSPF

## Issue encountered - NAT config was lost/never applied
- First attempt at `show ip nat translations` and `show ip nat 
  statistics` came back completely empty even after supposedly 
  configuring inside/outside interfaces and NAT rule
- Reapplied the entire NAT configuration cleanly from scratch 
  (inside/outside designation, ACL, overload rule) - this time 
  `show ip nat statistics` correctly showed the configured 
  interfaces and ACL

## Verification - NAT/PAT CONFIRMED WORKING
- PC-IT1 (192.168.20.10) successfully pinged 8.8.4.1 (ISP-RTR's 
  outside interface, simulating the internet) - 0% packet loss
- `show ip nat translations` confirmed real-time PAT entries:
  192.168.20.10:13 -> 203.0.113.1:13 (and similar for each ping)
- Confirms multiple internal devices could share the single public 
  IP (203.0.113.1) simultaneously, tracked by port number

## Key concept confirmed
NAT/PAT allows an entire private internal network (multiple VLANs, 
potentially thousands of devices) to share a single public-facing 
IP address to reach the internet - this is exactly how real-world 
home and business networks operate. A default route (not a routing 
protocol advertisement) is the correct way to point internal 
networks toward "the internet" as a whole, since routing protocols 
like OSPF are for known/internal topology only.

## Next phase
- Set up DHCP on HQ-DHCP-DNS server (replace static IPs on PCs)
- Configure ACLs for Guest VLAN restriction
