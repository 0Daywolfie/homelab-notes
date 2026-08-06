# Phase 12: Wireless (WLC + AP) - IN PROGRESS

## What was completed tonight
- Added HQ-WLC (Wireless LAN Controller) to the topology, cabled to 
  HQ-CORE
- Created VLAN 60 (Wireless) on HQ-CORE with SVI 192.168.60.1/24, 
  ip helper-address pointing to HQ-RTR, added to OSPF advertisement
- Added an AP device to the topology
- Assigned Fa0/6 on HQ-ACC1 to VLAN 60, renamed the VLAN locally 
  to "Wireless" (was showing auto-generated "VLAN0060")

## Open issue - AP cabling unclear, not yet resolved
show interfaces status on HQ-ACC1 shows Fa0/6 as "connected" in 
VLAN 60, but on visual inspection there does not appear to be an 
actual physical cable linking the AP to that port. Needs 
investigation next session:
- Click the actual cable/wire on canvas to confirm real endpoints
- Check the AP device's own Config/Physical tab for its actual 
  connected port status
- Possible causes: AP may be cabled to a different device entirely, 
  Fa0/6's "connected" status may be stale/leftover from earlier 
  testing (that port was reused multiple times tonight for NAT 
  testing and wireless), or the AP genuinely isn't cabled yet

## Still to do once cabling is confirmed
- Set HQ-WLC's IP: 192.168.10.40/24, gateway 192.168.10.1
- Configure SSID and WPA2 security on the controller
- Confirm AP registers with the WLC
- Test wireless client association and connectivity through VLAN 60

## Next session - START HERE
1. Resolve the AP cabling confusion (see Open issue above)
2. Complete WLC IP addressing and SSID/security configuration
3. Test wireless client connectivity
4. Second branch site
5. IoT segment
