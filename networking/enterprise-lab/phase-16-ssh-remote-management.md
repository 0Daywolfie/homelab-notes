# Phase 16: SSH Remote Management + MOTD Banners - VERIFIED WORKING

## Background
Added a NOC-ADMIN PC to VLAN 10 (Management) to explore the 
network, which led to discovering and fixing a real NAT ACL 
regression (Phase 15). Continued exploring by asking whether 
topology mapping (via CDP) could be done remotely from the PC 
rather than physically clicking into each device.

## Why SSH instead of Telnet
Chose SSH over Telnet deliberately - Telnet transmits credentials 
and session data in plaintext, SSH encrypts the full session. This 
is standard practice in real networks; Telnet is generally 
considered a security liability today.

## What was configured (applied to every router and switch)
    ip domain-name homelab.local
    username admin privilege 15 secret ChiefAdmin2026!
    crypto key generate rsa    (1024-bit)

    line vty 0 4
    transport input ssh
    login local

    banner motd #
    [warning/identification message]
    #

Applied to: HQ-CORE, HQ-RTR, BR-RTR, BR2-RTR, HQ-ACC1, HQ-ACC2, 
BR-SW, BR2-SW.

## Key concept - SSH prerequisites
Unlike Telnet (which can run off just a line password), SSH 
requires: a domain name (for key generation), RSA crypto keys, and 
typically local username/password authentication rather than a 
shared line password. transport input ssh explicitly disables 
Telnet on the VTY lines rather than just adding SSH alongside it.

## Verification - CONFIRMED WORKING
From NOC-ADMIN's command line:
    ssh -l admin 192.168.10.1
Successfully connected to HQ-CORE remotely, over an encrypted 
session, from a completely separate device on the network - not 
console access, genuine remote management. Custom MOTD banner 
displayed correctly before the password prompt.

Repeated across all routers and switches in the topology - NOC-ADMIN 
can now remotely and securely manage every network device from a 
single seat, matching how a real NOC operator works.

## What this enables going forward
CDP-based topology mapping (the original goal that led here) can 
now be done entirely from NOC-ADMIN by SSHing into each device in 
turn and running show cdp neighbors, rather than physically 
clicking into every device in Packet Tracer - genuinely mirrors 
how real engineers explore unfamiliar networks remotely.

## Next
- Use SSH sessions from NOC-ADMIN to walk the network via CDP and 
  build a topology map from a single vantage point
