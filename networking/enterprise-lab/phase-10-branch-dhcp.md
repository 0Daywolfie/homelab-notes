# Phase 10: Branch DHCP - VERIFIED WORKING

## What was configured
Added a DHCP pool for Branch's VLAN 50 on HQ-RTR, matching the 
pattern used for HQ's VLANs:

    HQ-RTR:
    ip dhcp pool BRANCH-POOL
    network 192.168.50.0 255.255.255.0
    default-router 192.168.50.1
    dns-server 192.168.10.10

    ip dhcp excluded-address 192.168.50.1

## Key design difference from HQ's DHCP relay
At HQ, ip helper-address was needed on HQ-CORE's SVIs because the 
Layer 3 gateway (SVI) and the DHCP pool (HQ-RTR) live on different 
devices.

At Branch, the gateway IS the DHCP-relaying device (BR-RTR's Gig0/0 
serves both roles) - but the pool still lives on HQ-RTR across the 
WAN link, so relay was still required, just configured on a 
different device than at HQ:

    BR-RTR:
    interface gigabitEthernet 0/0
    ip helper-address 192.168.100.2

Reasoned through this rather than copying the HQ pattern blindly - 
confirmed the helper-address needs to point at whichever device 
actually holds the DHCP pool (HQ-RTR), reachable via the already-
verified OSPF routing between sites.

## Verification - CONFIRMED WORKING
Both PC-BR1 and PC-BR2 successfully obtained DHCP leases in the 
192.168.50.0/24 range with correct gateway (192.168.50.1) after 
switching from static to DHCP mode.

## Lab status - fully dynamic addressing achieved
Every end-user VLAN across both HQ and Branch now uses DHCP - IT, 
Sales, Finance, Guest (HQ) and Branch. No static IP configuration 
remains on any end-user device anywhere in the topology.

## Next phase
- Second branch site
- NOC - centralized monitoring
- Wireless - WLC + AP
- IoT segment
