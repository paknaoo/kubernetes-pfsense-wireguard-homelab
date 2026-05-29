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
- controlled Internet egress for VPN clients through OPT1

## Interfaces

| Interface | Address | Purpose |
|------|------|------|
| WAN / OUTSIDE | `192.168.50.254` | External / hypervisor-facing network |
| LAN | `10.10.10.254` | Kubernetes cluster network |
| WireGuard | `10.20.20.1` | VPN tunnel endpoint |
| OPT1 | DHCP | Internet uplink for VPN egress |

## Default Gateway

pfSense uses the OPT1 DHCP gateway as the default gateway for Internet egress.

| Setting | Value |
|------|------|
| Default gateway | `OPT1_DHCP` |

This ensures full-tunnel VPN Internet traffic exits through the real Internet uplink rather than the lab OUTSIDE network.

## DHCP Static Mappings

Static DHCP mappings are used for Kubernetes nodes to keep cluster addressing predictable.

| Host | Address |
|------|------|
| `k8s-master` | `10.10.10.10` |
| `worker1` | `10.10.10.11` |
| `worker2` | `10.10.10.12` |
| `worker3` | `10.10.10.13` |

## Firewall Model

The firewall model follows a least-privilege approach for internal LAN access.

WAN / OUTSIDE exposure is limited to WireGuard access from the management workstation.

Internet egress from the WireGuard subnet is handled separately through controlled firewall rules and outbound NAT via OPT1.

### WAN / OUTSIDE Rule

| Source | Destination | Protocol | Purpose |
|------|------|------|------|
| `192.168.50.10` | pfSense WAN address | UDP 51820 | WireGuard VPN access |

### VPN Internet Egress

WireGuard clients are permitted to use controlled outbound Internet access through pfSense OPT1.

| Source | Destination | Protocol / Ports | Purpose |
|------|------|------|------|
| `10.20.20.0/24` | any | TCP/UDP 53 | DNS |
| `10.20.20.0/24` | any | TCP 80 | HTTP |
| `10.20.20.0/24` | any | TCP 443 | HTTPS |
| `10.20.20.0/24` | any | UDP 123 | NTP |
| `10.20.20.0/24` | any | ICMP | Optional connectivity testing |

VPN clients remain restricted to approved internal management targets through pfSense firewall policy and aliases.

## Outbound NAT

Outbound NAT is required for WireGuard client traffic to reach the Internet through OPT1.

The lab uses Hybrid Outbound NAT with a rule for the WireGuard subnet.

| Interface | Source | Destination | Translation |
|------|------|------|------|
| OPT1 | `10.20.20.0/24` | any | OPT1 address |

Without this NAT rule, WireGuard access to internal LAN resources works, but Internet egress from the VPN client fails.

## Alias Model for Internal Access

pfSense aliases are used to keep internal VPN access rules readable and easier to manage.

| Alias | Value | Purpose |
|------|------|------|
| `WG_CLIENTS` | `10.20.20.2` | WireGuard management client |
| `K8S_ADMIN` | `10.10.10.10` | Kubernetes control plane |
| `K8S_INGRESS` | `10.10.10.50` | Envoy Gateway / MetalLB ingress |
| `PFSENSE_GUI` | `10.10.10.254` | pfSense LAN GUI |
| `PFSENSE_WG` | `10.20.20.1` | pfSense WireGuard tunnel address |
| `MGMT_ALLOWED` | approved management aliases | Allowed VPN destinations |

## VPN Access Policy

The WireGuard firewall policy restricts internal LAN access to approved management destinations.

Internet egress is allowed separately through controlled OPT1 firewall rules and outbound NAT.

Allowed from VPN:

- pfSense LAN GUI
- Kubernetes control plane
- Envoy Gateway / MetalLB ingress
- pfSense WireGuard tunnel address

Blocked from VPN:

- Kubernetes worker nodes
- remaining LAN hosts
- broad LAN access
