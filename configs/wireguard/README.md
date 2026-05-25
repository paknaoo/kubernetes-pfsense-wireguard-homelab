# WireGuard Configuration

This directory contains a sanitised WireGuard client configuration example used for remote administration of the lab.

The real private keys and endpoint details are intentionally not included.

## Purpose

WireGuard provides the primary remote administration path into the lab environment.

The VPN client is configured with restricted `AllowedIPs` so that it can reach only approved management targets rather than the full LAN network.

## Allowed Routes

| Route | Purpose |
|------|------|
| `10.10.10.10/32` | Kubernetes control plane |
| `10.10.10.50/32` | Envoy Gateway / MetalLB ingress |
| `10.10.10.254/32` | pfSense LAN GUI |
| `10.20.20.0/24` | WireGuard VPN subnet |

## Files

| File | Purpose |
|------|------|
| `wg0.conf.example` | Sanitised WireGuard client configuration example |

## Notes

The example configuration demonstrates the restricted routing model used in the lab.

Secrets such as private keys, public keys, and real endpoint values are replaced with placeholders.
