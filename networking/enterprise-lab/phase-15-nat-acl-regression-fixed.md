# Phase 15: NAT Access-List Regression - Found and Fixed

## Background
Added a NOC-ADMIN PC to VLAN 10 (Management) purely to explore and 
test reachability across the whole network - not part of the 
planned roadmap, just genuine curiosity about what the built 
network could actually do.

## What was found
While testing internet access from NOC-ADMIN, discovered 
access-list 1 (used by NAT) only contained 3 entries: 
192.168.50.0/24, 192.168.51.0/24, 192.168.70.0/24 (Branch 1, 
Branch 2, IoT - the most recently added subnets).

Entries for VLANs 10 (Management), 20 (IT), 30 (Sales), 40 
(Finance), 60 (Wireless), and 99 (Guest) were missing entirely - 
despite having been added and verified working in earlier phases.

Confirmed this was a genuine functional regression, not just a 
stale display - tested directly from PC-IT1:
    ping 8.8.4.1
Result: 100% packet loss, confirming IT had actually lost internet 
access at some point during later session work (likely during 
IoT or Branch 2 configuration - exact cause not identified, 
possibly an interrupted or overwritten access-list command).

## Fix
Rebuilt the complete access-list with all 9 entries:
    access-list 1 permit 192.168.10.0 0.0.0.255
    access-list 1 permit 192.168.20.0 0.0.0.255
    access-list 1 permit 192.168.30.0 0.0.0.255
    access-list 1 permit 192.168.40.0 0.0.0.255
    access-list 1 permit 192.168.50.0 0.0.0.255
    access-list 1 permit 192.168.51.0 0.0.0.255
    access-list 1 permit 192.168.60.0 0.0.0.255
    access-list 1 permit 192.168.70.0 0.0.0.255
    access-list 1 permit 192.168.99.0 0.0.0.255

Also deliberately included Management (192.168.10.0/24) for the 
first time, giving NOC-ADMIN internet access.

## Verification - CONFIRMED FIXED
- PC-IT1 -> 8.8.4.1: SUCCESS (internet access restored)
- NOC-ADMIN -> 8.8.4.1: SUCCESS (Management now has internet access)

## Bonus finding - Management VLAN reachability
Before the fix was even needed, confirmed NOC-ADMIN (Management 
VLAN) could reach every other VLAN's gateway across the entire 
network - IT, Sales, Branch 1, Branch 2, Wireless, IoT, and Guest 
all responded successfully. This confirms the asymmetric security 
design is working as intended: Guest and IoT are blocked FROM 
reaching other VLANs (via their respective restriction ACLs), but 
nothing blocks Management FROM reaching everywhere - which is 
correct, since Management/admin access needs full network 
visibility to actually manage the network.

## Key lesson
Long configuration sessions with many changes across many devices 
can silently regress earlier working config, especially on shared 
resources like a single access-list that multiple phases touch 
over time. Periodically re-verifying previously-working 
functionality (not just testing what you just built) catches this 
kind of drift. This was found purely through exploratory testing, 
not a scheduled check - worth doing periodically even after a 
build is "done."
