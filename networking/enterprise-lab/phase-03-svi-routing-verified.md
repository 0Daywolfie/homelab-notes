# Phase 3: SVI Configuration and Inter-VLAN Routing - VERIFIED WORKING

## What was configured
- Enabled `ip routing` on HQ-CORE
- Created 5 SVIs on HQ-CORE (Vlan10, 20, 30, 40, 99) with gateway IPs:
  - Vlan10: 192.168.10.1/24
  - Vlan20: 192.168.20.1/24
  - Vlan30: 192.168.30.1/24
  - Vlan40: 192.168.40.1/24
  - Vlan99: 192.168.99.1/24
- Assigned static IPs to 6 HQ PCs matching their VLAN subnets
- Set default gateway on each PC to match its VLAN's SVI

## Issue encountered
- Vlan20 SVI initially didn't get created (command block got 
  interrupted) - had to add it separately after noticing it missing 
  from `show ip interface brief`

## Verification - INTER-VLAN ROUTING CONFIRMED WORKING
- PC-IT1 (192.168.20.10, VLAN 20) -> PC-SALES1 (192.168.30.10, VLAN 30): 
  SUCCESS (0% loss after initial ARP timeout)
- PC-IT1 -> HQ-CORE Vlan10 SVI (192.168.10.1): SUCCESS, 0% loss, TTL=255
- PC-IT1 -> PC-FIN1 (192.168.40.10, VLAN 40): SUCCESS (0% loss after 
  initial ARP timeout)
- First ping in each test showed "Request timed out" - normal ARP 
  learning behavior, not an actual problem

## Key concept confirmed
HQ-CORE (Layer 3 switch) is successfully routing traffic between 
VLANs using SVIs, with no separate router required for inter-VLAN 
routing. This is real enterprise-pattern design, not router-on-a-stick.

## Next phase
- Configure OSPF between HQ-RTR and BR-RTR for site-to-site routing
- Configure NAT/PAT on HQ-RTR for internet access via ISP-RTR
- Set up DHCP on HQ-DHCP-DNS server (replace static IPs)
- Configure ACLs for Guest VLAN restriction
