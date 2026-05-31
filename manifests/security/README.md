# Kubernetes Security Manifests

This directory contains Kubernetes security manifests used to validate NetworkPolicies and RBAC hardening in the lab.

The goal is to demonstrate Kubernetes-native least-privilege controls at both the network and API authorisation layers.

## Structure

| Directory | Purpose |
|------|------|
| `network-policies/` | Calico NetworkPolicies for default-deny and explicit pod communication |
| `rbac/` | ServiceAccounts, Roles, RoleBindings, and ClusterRoleBindings for API access control |

## Security Controls

| Layer | Control | Purpose |
|------|------|------|
| Network | NetworkPolicies | Restrict pod-to-pod communication |
| Authorisation | RBAC | Restrict Kubernetes API permissions |

## NetworkPolicy Model

NetworkPolicies are used in the `security-lab` namespace to enforce least-privilege pod communication.

Implemented behaviour:

| Flow | Result |
|------|------|
| Default pod-to-pod traffic | Denied |
| DNS egress | Explicitly allowed |
| `backend → frontend` | Allowed |
| `attacker → frontend` | Denied |

The model validates default-deny networking with explicit allow rules for required application traffic.

## RBAC Model

RBAC manifests validate least-privilege Kubernetes API access.

Implemented identities:

| Identity | Scope | Purpose |
|------|------|------|
| `readonly-user` | Cluster-wide read-only | Allows read access without write permissions |
| `demo-admin` | `demo` namespace | Allows admin access only within one namespace |
| `demo-deployer` | `demo` namespace | Allows limited deployment operations without secrets or RBAC modification |

## Validation

NetworkPolicy validation uses positive and negative connectivity tests between test pods.

RBAC validation uses:

```bash
kubectl auth can-i
```

Example checks include:

```bash
kubectl auth can-i get pods \
  --as=system:serviceaccount:rbac-lab:readonly-user \
  -A
```

```bash
kubectl auth can-i get secrets \
  --as=system:serviceaccount:demo:demo-deployer \
  -n demo
```

## Notes

These manifests are intended for lab validation and documentation.

They demonstrate least-privilege Kubernetes security controls using dedicated test namespaces and service accounts.

Detailed design and validation notes are documented in [`docs/kubernetes-security.md`](../../docs/kubernetes-security.md).
