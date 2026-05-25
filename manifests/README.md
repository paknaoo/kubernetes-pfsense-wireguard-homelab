# Kubernetes Manifests

This directory contains Kubernetes manifests used to deploy and validate key platform components in the lab.

The manifests are organised by component to keep the repository easy to review and reuse.

## Structure

| Directory | Purpose |
|------|------|
| `metallb/` | MetalLB address pool and Layer 2 advertisement |
| `envoy-gateway/` | Gateway API, Envoy Gateway routing, and test backend |
| `metrics-server/` | Lab-specific metrics-server adjustment and validation notes |
| `storage/` | Persistent storage validation using local-path-provisioner |


## Notes

These manifests are intended to document and reproduce selected lab configuration, not to fully automate the entire cluster build.

Cluster bootstrap, pfSense configuration, and WireGuard configuration are documented separately.
