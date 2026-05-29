# WireGuard Configuration

This directory contains a sanitised WireGuard client configuration example used for remote administration of the lab.

The real private keys and endpoint details are intentionally not included.

## Purpose

WireGuard provides the primary remote administration path into the lab and supports full-tunnel Internet egress through pfSense.

The VPN client uses full-tunnel routing so that Internet traffic is routed through pfSense and exits through the OPT1 uplink.

Internal LAN access remains restricted by pfSense firewall policy and aliases.

## Routing Model

| Setting | Value |
|------|------|
| Client VPN address | `10.20.20.2/24` |
| pfSense WireGuard address | `10.20.20.1/24` |
| AllowedIPs | `0.0.0.0/0` |
| DNS | `10.10.10.254` |
| Internet egress | pfSense OPT1 uplink |

## Internal Access Control

Although the WireGuard client uses full-tunnel routing, internal LAN access is restricted separately by pfSense firewall policy.

Allowed internal targets include:

| Target | Purpose |
|------|------|
| `10.10.10.10` | Kubernetes control plane |
| `10.10.10.50` | Envoy Gateway / MetalLB ingress |
| `10.10.10.254` | pfSense LAN GUI |

## Files

| File | Purpose |
|------|------|
| [`wg0.conf.example`](wg0.conf.example) | Sanitised WireGuard client configuration example using full-tunnel routing |

## Notes

The example configuration demonstrates the restricted routing model used in the lab.

Secrets such as private keys, public keys, and real endpoint values are replaced with placeholders.
