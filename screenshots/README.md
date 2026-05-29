# Screenshots

This directory contains selected validation screenshots from the Kubernetes, pfSense, and WireGuard homelab.

The screenshots provide visual proof of cluster health, metrics collection, ingress routing, VPN connectivity, and restricted firewall-based access control.

## Screenshots

| File | Purpose |
|------|------|
| [`kubectl-get-nodes.png`](kubectl-get-nodes.png) | Shows all Kubernetes nodes in `Ready` state |
| [`kubectl-top-nodes.png`](kubectl-top-nodes.png) | Shows metrics-server collecting node metrics |
| [`kubectl-top-pods.png`](kubectl-top-pods.png) | Shows pod metrics across namespaces |
| [`envoy-gateway-curl.png`](envoy-gateway-curl.png) | Shows successful Envoy Gateway / HTTPRoute validation using `curl` |
| [`envoy-gateway-resources.png`](envoy-gateway-resources.png) | Shows Gateway API and Envoy Gateway Kubernetes resources |
| [`wireguard-handshake.png`](wireguard-handshake.png) | Shows successful WireGuard VPN tunnel establishment |
| [`pfsense-aliases.png`](pfsense-aliases.png) | Shows pfSense aliases used for the least-privilege access model |
| [`pfsense-rules-wg.png`](pfsense-rules-wg.png) | Shows restricted WireGuard firewall policy on pfSense |
| [`wireguard-full-tunnel-route.png`](wireguard-full-tunnel-route.png) | Shows WireGuard full-tunnel policy routing on `mgmt01` |
| [`pfsense-outbound-nat-opt1.png`](pfsense-outbound-nat-opt1.png) | Shows outbound NAT translating the WireGuard subnet through OPT1 |
| [`pfsense-opt1-gateway.png`](pfsense-opt1-gateway.png) | Shows pfSense using the OPT1 DHCP gateway for Internet egress |
| [`vpn-internet-egress-curl.png`](vpn-internet-egress-curl.png) | Shows Internet access from `mgmt01` through the WireGuard tunnel |

## Notes

This is a non-production lab environment using private RFC1918 addressing.

Internal lab IP addresses are intentionally shown in screenshots where they help explain the network design and validation results.

Private keys, secrets, and full configuration exports are not included, as they are not required to understand the project and excluding them reflects good security practice.
