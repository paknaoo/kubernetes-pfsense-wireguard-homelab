# pfSense Configuration Notes

This directory documents the pfSense configuration model used in the lab.

## Role

pfSense provides the core network boundary for the lab environment.

It is responsible for:

- routing between lab networks
- LAN DHCP services
- firewall policy enforcement
- WireGuard VPN termination
- controlled administrative access

## Interfaces

| Interface | Address | Purpose |
|------|------|------|
| WAN / OUTSIDE | `192.168.50.254` | External / hypervisor-facing network |
| LAN | `10.10.10.254` | Kubernetes cluster network |
| WireGuard | `10.20.20.1` | VPN tunnel endpoint |

## DHCP Static Mappings

Static DHCP mappings are used for Kubernetes nodes to keep cluster addressing predictable.

| Host | Address |
|------|------|
| `k8s-master` | `10.10.10.10` |
| `worker1` | `10.10.10.11` |
| `worker2` | `10.10.10.12` |
| `worker3` | `10.10.10.13` |

## Firewall Model

The firewall model follows a least-privilege approach.

WAN exposure is limited to WireGuard access from the management workstation.

| Source | Destination | Protocol | Purpose |
|------|------|------|------|
| `192.168.50.10` | pfSense WAN | UDP 51820 | WireGuard VPN access |

VPN clients are restricted to approved management targets through pfSense firewall policy and aliases.

## Alias Model

| Alias | Value | Purpose |
|------|------|------|
| `WG_CLIENTS` | `10.20.20.2` | WireGuard management client |
| `K8S_ADMIN` | `10.10.10.10` | Kubernetes control plane |
| `K8S_INGRESS` | `10.10.10.50` | Envoy Gateway / MetalLB ingress |
| `PFSENSE_GUI` | `10.10.10.254` | pfSense LAN GUI |
| `PFSENSE_WG` | `10.20.20.1` | pfSense WireGuard tunnel address |
| `MGMT_ALLOWED` | approved management aliases | Allowed VPN destinations |

## VPN Access Policy

The WireGuard firewall policy permits VPN clients to access only approved management destinations.

Allowed from VPN:

- pfSense LAN GUI
- Kubernetes control plane
- Envoy Gateway / MetalLB ingress
- pfSense WireGuard tunnel address

Blocked from VPN:

- Kubernetes worker nodes
- remaining LAN hosts
- broad LAN access
