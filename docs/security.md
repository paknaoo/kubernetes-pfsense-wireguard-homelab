# Security Model

## Overview

The lab follows a least-privilege security model centred around pfSense firewalling, WireGuard VPN access, and controlled administrative reachability.

Remote administration is intentionally restricted to approved management targets, while unnecessary exposure between networks and systems is avoided.

The design focuses on:

- minimal WAN exposure
- explicit allow rules
- segmented network boundaries
- restricted VPN routing
- controlled administrative access

