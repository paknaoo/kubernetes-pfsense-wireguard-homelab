# NetworkPolicy Manifests

This directory contains Kubernetes NetworkPolicy manifests used to validate least-privilege pod-to-pod communication in the `security-lab` namespace.

The policies are designed for Calico and demonstrate a default-deny model with explicit allow rules for DNS and required application traffic.

## Files

| File | Purpose |
|------|------|
| [`default-deny.yaml`](default-deny.yaml) | Denies ingress and egress traffic for all pods in `security-lab` unless explicitly allowed |
| [`allow-dns.yaml`](allow-dns.yaml) | Allows DNS egress to CoreDNS in `kube-system` |
| [`allow-backend-to-frontend.yaml`](allow-backend-to-frontend.yaml) | Allows ingress to `frontend` from pods labelled `run=backend` |
| [`allow-backend-egress-to-frontend.yaml`](allow-backend-egress-to-frontend.yaml) | Allows egress from `backend` to pods labelled `run=frontend` |

## Test Workloads

The validation model used three test pods:

| Pod | Purpose |
|------|------|
| `frontend` | nginx test application |
| `backend` | allowed client pod |
| `attacker` | denied client pod for negative validation |

## Expected Behaviour

| Flow | Expected Result |
|------|------|
| `backend → frontend` | Allowed |
| `attacker → frontend` | Denied |
| DNS egress | Allowed |
| Unspecified pod traffic | Denied |

## Notes

NetworkPolicies are label-driven. The lab workloads use the default labels created by `kubectl run`:

```text
run=frontend
run=backend
```

Use the following command when validating labels:

```bash
kubectl get pods -n security-lab --show-labels
```

Detailed design and validation notes are documented in [`docs/kubernetes-security.md`](../../../docs/kubernetes-security.md).
