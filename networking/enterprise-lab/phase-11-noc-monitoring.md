# Phase 11: NOC - Centralized Monitoring (Syslog + NTP) - VERIFIED WORKING

## What was built
Added a NOC-SYSLOG server (192.168.10.30) in VLAN 10 (Management), 
alongside the existing infrastructure servers.

## Syslog - centralized logging
Enabled the Syslog service on NOC-SYSLOG, then pointed every network 
device at it:

    [hostname]#configure terminal
    [hostname](config)#logging host 192.168.10.30
    [hostname](config)#exit
    [hostname]#write memory

Applied to HQ-RTR, HQ-CORE, HQ-ACC1, HQ-ACC2, BR-RTR, BR-SW.

## Issue encountered - logging trap syntax not supported
Attempted to set logging trap informational for a specific severity 
level. This IOS image only supports logging trap debugging or no 
argument at all (checked via logging trap ?). Skipped setting a 
custom trap level entirely - logging host alone was sufficient to 
get logs flowing to the server with this IOS image's default behavior.

## Verification - Syslog CONFIRMED WORKING
Triggered a real event (shut/no shut on HQ-ACC1's Fa0/1) and 
confirmed live log entries appeared on NOC-SYSLOG's Syslog service 
screen, correctly showing the source device's IP (10.0.0.2 for 
HQ-RTR's WAN interface, 192.168.10.1 for HQ-CORE's Vlan10 SVI) and 
%SYS-6 level messages.

## NTP - synchronized time across all devices
Enabled the NTP service on NOC-SYSLOG, then pointed every device at 
it as their time reference:

    [hostname]#configure terminal
    [hostname](config)#ntp server 192.168.10.30
    [hostname](config)#exit
    [hostname]#write memory

Applied to the same six devices as syslog.

## Verification - NTP CONFIRMED WORKING
show ntp status on HQ-RTR and HQ-CORE both confirm:
"Clock is synchronized, stratum 2, reference is 192.168.10.30"

Both devices now share a consistent, synchronized clock source - 
meaning log timestamps across the entire network are now genuinely 
comparable during troubleshooting, not just locally accurate to 
each device's own independent clock.

## Key concept confirmed
A real NOC setup centers on two core pillars before any dashboards 
or alerting matter: centralized logging (so you're not manually 
checking every device) and synchronized time (so those logs can 
actually be correlated across devices during an incident). Without 
NTP, syslog timestamps from different devices are not directly 
comparable, undermining the whole point of centralizing logs in 
the first place.

## Next phase
- Second branch site
- Wireless - WLC + AP
- IoT segment
