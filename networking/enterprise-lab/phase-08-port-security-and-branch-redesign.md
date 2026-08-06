# Phase 8: Port Security + Branch Subnet Redesign

## Port Security - implemented across all access switches
Used interface range instead of configuring ports one at a time:

HQ-ACC1 (Fa0/1-4):
    interface range fastEthernet 0/1 - 4
    switchport port-security
    switchport port-security maximum 1
    switchport port-security mac-address sticky
    switchport port-security violation shutdown

Same pattern applied to HQ-ACC2 (Fa0/1-2) and BR-SW (Fa0/1-2).

Note: sticky learning only captures a MAC the next time that port 
sees traffic after port-security is enabled - does not retroactively 
learn already-idle connected devices. Had to trigger a ping from 
each PC to force the switch to learn and lock the MAC address.

Verified via show port-security and show port-security address on 
each switch - all ports showed CurrentAddr: 1 after triggering 
traffic.

## Design flaw found and fixed - Branch subnet too small for PCs
While setting up port security on BR-SW, discovered PC-BR1 and 
PC-BR2 had no way to get valid IPs. Root cause: the original WAN 
addressing scheme used 192.168.101.0/30 for the BR-RTR-to-BR-SW 
link, treating it like a point-to-point WAN transit network. A /30 
only has 2 usable addresses, both already consumed by BR-RTR's own 
interface - there was never room for actual PCs in that subnet.

## Fix - redesigned Branch to use a proper /24 LAN subnet
Changed 192.168.101.0/30 to 192.168.50.0/24:

    BR-RTR:
    interface gigabitEthernet 0/0
    ip address 192.168.50.1 255.255.255.0

    router ospf 1
    no network 192.168.101.0 0.0.0.3 area 0
    network 192.168.50.0 0.0.0.255 area 0

    HQ-RTR (NAT access-list needed updating too):
    no access-list 1 permit 192.168.101.0 0.0.0.3
    access-list 1 permit 192.168.50.0 0.0.0.255

Verified via show ip route on HQ-RTR - 192.168.50.0/24 now shows 
up correctly learned via OSPF, old /30 entry is gone.

PC-BR1 assigned 192.168.50.10/24, PC-BR2 assigned 192.168.50.11/24, 
gateway 192.168.50.1.

## Open issue - not yet resolved
After the subnet redesign, PC-BR1 still fails to ping its own 
gateway (192.168.50.1), the internet (8.8.4.1), or HQ (192.168.20.1) 
- all requests time out, despite:
- BR-RTR's interface and routing table showing everything correct
- Port security confirmed NOT the cause (checked show port-security, 
  zero violations, port shows connected)
- Waited for simulation to settle, retried - still failing

Suspected: ARP cache staleness after changing the subnet under a 
live interface, or another Packet Tracer simulation lag similar to 
the DHCP/ACL display issues hit in Phase 7. Flagged for further 
investigation next session - possible next steps: clear ARP cache 
on BR-RTR, reload BR-RTR, or rebuild the PC's network config from 
scratch.

## Key lesson
Point-to-point WAN link subnets (/30) and LAN subnets hosting end 
devices serve different purposes and need very different sizing. 
Reusing a /30 designed for a router-to-router link to also host 
end-user PCs is a common design mistake - always separate WAN 
transit addressing from LAN host addressing.

## Next session - START HERE
1. Resolve PC-BR1/PC-BR2 connectivity issue (see Open issue above)
2. Multi-area OSPF - split into Area 0 (HQ) and Area 1 (Branch), 
   HQ-RTR becomes an ABR
3. Second branch site
4. NOC - centralized monitoring (syslog, NTP, SNMP concepts)
5. Wireless - WLC + AP
6. IoT segment - camera/smart devices on isolated VLAN
