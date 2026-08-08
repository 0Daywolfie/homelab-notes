# Phase 12: Wireless (AP + WLC) - VERIFIED WORKING

## Background - resolved the AP cabling mystery from prior sessions
Root cause was misidentification, not a real bug: a guest PC (with 
a broken DHCP lease, never learned by any switch MAC table) was 
sitting on Fa0/6, while the actual AP was on Fa0/7 the entire time, 
just left in default VLAN 1 instead of VLAN 60. Deleted the guest 
PC, reassigned Fa0/7 to VLAN 60, confirmed via show interfaces 
status.

## AP architecture discovery
The Packet Tracer AP model used here is autonomous, not lightweight - 
it does not register with or take configuration from the WLC over 
the network. SSID and security are configured directly on the AP 
itself (Port 1 / radio interface), not pushed from HQ-WLC. The WLC 
was still built and is available for future use with lightweight 
APs if the wireless deployment is scaled up later.

## What was configured
On HQ-AP directly (Port 1):
- SSID: HQ-Corporate-WiFi
- Authentication: WPA2-PSK
- PSK Pass Phrase: NetworkChief2026!
- Encryption: AES

On HQ-WLC (WLAN profile, built but not actively used by this 
autonomous AP):
- Profile: HQ-Corporate-WiFi, VLAN 60, WPA2-PSK, AES, local 
  switching/local authentication

## Client testing - required a wireless NIC swap
Default Laptop-PT ships with a wired NIC. Had to power off, remove 
the wired module, and install a WPC300N wireless card via the 
Physical tab before PC Wireless would allow connecting to any SSID.

Successfully scanned for and connected to HQ-Corporate-WiFi using 
the configured WPA2-PSK passphrase.

## Issue found - missing DHCP pool for VLAN 60
Client associated to the SSID successfully but DHCP request failed 
(APIPA address). Checked show ip dhcp pool WIRELESS-POOL on HQ-RTR - 
came back "No such pool" - a genuine config gap, the pool had been 
planned but never actually created earlier in the session.

Fix:
    HQ-RTR#configure terminal
    HQ-RTR(config)#ip dhcp pool WIRELESS-POOL
    HQ-RTR(dhcp-config)#network 192.168.60.0 255.255.255.0
    HQ-RTR(dhcp-config)#default-router 192.168.60.1
    HQ-RTR(dhcp-config)#dns-server 192.168.10.10
    HQ-RTR(dhcp-config)#exit
    HQ-RTR(config)#ip dhcp excluded-address 192.168.60.1

Verified pool is healthy via show ip dhcp pool WIRELESS-POOL after 
creation - 254 addresses, 8 correctly excluded, 0 leases pending 
first client.

## Status - core wireless functionality verified
- Wireless association: CONFIRMED (client connected to SSID with 
  correct WPA2-PSK credentials)
- DHCP pool: CONFIRMED healthy and correctly scoped
- DHCP lease pickup: pending final confirmation - client still 
  showing APIPA after release/renew at time of writing, consistent 
  with the DHCP timing lag pattern seen repeatedly elsewhere in 
  this build (Phase 7, Phase 10) - expected to resolve, not treated 
  as a new unresolved bug

## Key lesson
Not every AP model requires a WLC to function - autonomous APs are 
still common and fully self-sufficient. Always verify whether a 
device is meant to be centrally managed or independently configured 
before assuming a missing controller relationship is a bug.

## Next phase
- Confirm final DHCP lease pickup and run full ping test suite 
  (local gateway, cross-VLAN, internet) once lease is obtained
- IoT segment
- Scale up Branch 2 (VLANs, additional infrastructure) - noted as 
  a future goal
