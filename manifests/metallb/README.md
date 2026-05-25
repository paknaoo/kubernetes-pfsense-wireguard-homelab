# MetalLB Manifests

This directory contains the MetalLB configuration used to provide `LoadBalancer` service support inside the Kubernetes lab.

MetalLB allocates service addresses from the LAN network so that Kubernetes services can be exposed without relying on a cloud provider load balancer.

## Address Pool

| Resource | Value |
|------|------|
| Address pool | `10.10.10.50-10.10.10.60` |
| Namespace | `metallb-system` |
| Current ingress address | `10.10.10.50` |

## Files

| File | Purpose |
|------|------|
| `ip-address-pool.yaml` | Defines the available MetalLB address range |
| `l2-advertisement.yaml` | Advertises the address pool on the LAN using Layer 2 mode |
