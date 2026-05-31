# RBAC Manifests

This directory contains Kubernetes RBAC manifests used to validate least-privilege Kubernetes API access in the lab.

The manifests demonstrate read-only access, namespace-scoped administration, and a custom deployer role with restricted permissions.

## Files

| File | Purpose |
|------|------|
| [`readonly-user.yaml`](readonly-user.yaml) | Creates a read-only ServiceAccount using the built-in `view` ClusterRole |
| [`demo-admin.yaml`](demo-admin.yaml) | Creates a namespace-scoped admin ServiceAccount for the `demo` namespace |
| [`demo-deployer-role.yaml`](demo-deployer-role.yaml) | Creates a custom least-privilege deployer Role and RoleBinding |

## Access Models

| Identity | Scope | Intended Access |
|------|------|------|
| `readonly-user` | Cluster-wide read-only | Inspect resources without modifying them |
| `demo-admin` | `demo` namespace | Administer resources only inside `demo` |
| `demo-deployer` | `demo` namespace | Deploy workloads without secrets or RBAC modification access |

## Validation

RBAC permissions can be validated using:

```bash
kubectl auth can-i
```

Example allowed checks:

```bash
kubectl auth can-i get pods \
  --as=system:serviceaccount:rbac-lab:readonly-user \
  -A
```

```bash
kubectl auth can-i create deployments \
  --as=system:serviceaccount:demo:demo-deployer \
  -n demo
```

Example denied checks:

```bash
kubectl auth can-i get secrets \
  --as=system:serviceaccount:demo:demo-deployer \
  -n demo
```

```bash
kubectl auth can-i create rolebindings \
  --as=system:serviceaccount:demo:demo-deployer \
  -n demo
```

```bash
kubectl auth can-i create deployments \
  --as=system:serviceaccount:demo:demo-deployer \
  -n security-lab
```

## Notes

The custom deployer role intentionally excludes access to:

- secrets
- RBAC resources
- cluster-wide resources
- other namespaces

Detailed design and validation notes are documented in [`docs/kubernetes-security.md`](../../../docs/kubernetes-security.md).
