# CGNAT / Hotspot Access Issue — Fixed with Tailscale

## Problem
Couldn't reach my home network remotely because my ISP/hotspot uses CGNAT, 
which blocks direct inbound connections.

## Solution
Set up Tailscale to create a private mesh VPN (tailnet), bypassing the CGNAT 
issue entirely by routing through Tailscale's relay infrastructure.

- Tailnet: kinti-hp-pavilion-14-notebook
- Verified SSH access from mobile over the tailnet

## Result
Reliable remote access to home machine regardless of network/CGNAT restrictions.
