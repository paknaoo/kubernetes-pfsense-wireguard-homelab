# Kubernetes Security

## Overview

This document describes Kubernetes-native security controls implemented in the lab using Calico NetworkPolicies and Kubernetes RBAC.

The goal is to enforce least privilege at two Kubernetes layers:

- pod-to-pod network communication
- Kubernetes API authorisation

The work was validated using dedicated test namespaces, positive and negative connectivity tests, and `kubectl auth can-i` checks.

## Contents

- [Security Scope](#security-scope)
- [NetworkPolicy Model](#networkpolicy-model)
- [RBAC Model](#rbac-model)
- [Validation](#validation)
- [Lessons Learned](#lessons-learned)

## Security Scope

Kubernetes security hardening was implemented using two control layers.

| Layer | Control | Purpose |
|------|------|------|
| Network | Calico NetworkPolicies | Restrict pod-to-pod communication |
| Authorisation | Kubernetes RBAC | Restrict Kubernetes API permissions |

Dedicated namespaces were used for validation:

| Namespace | Purpose |
|------|------|
| `security-lab` | NetworkPolicy testing |
| `rbac-lab` | Read-only RBAC identity testing |
| `demo` | Namespace-scoped admin and custom deployer RBAC testing |

## NetworkPolicy Model

NetworkPolicies were implemented to validate least-privilege pod communication using Calico.

The test model used:

- `frontend` pod running nginx
- `backend` pod running BusyBox
- `attacker` pod used for negative validation

### Baseline Connectivity

Before applying NetworkPolicies, baseline connectivity was confirmed between the backend client and frontend service.

```text
backend → frontend
```

Result:

```text
allowed
```

This confirmed that pod networking and service discovery were working before introducing policy restrictions.

### Default Deny

A namespace-wide default deny policy was applied in `security-lab`.

The policy selected all pods and denied both ingress and egress traffic unless explicitly allowed.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: security-lab
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

Observed behaviour:

| Flow | Result |
|------|------|
| Pod-to-pod traffic | Blocked |
| DNS egress | Blocked |
| Service access | Blocked |

This confirmed that default deny policies affect both application traffic and supporting cluster services such as DNS.

### DNS Allow Policy

An explicit DNS egress policy was added to allow pods in `security-lab` to resolve cluster service names.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: security-lab
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

Result:

| Capability | Result |
|------|------|
| DNS resolution | Restored |
| Application traffic | Still blocked |

### Explicit Backend to Frontend Access

Application traffic was then allowed explicitly from `backend` to `frontend`.

Ingress to the frontend pod was restricted to pods labelled `run=backend`.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-frontend
  namespace: security-lab
spec:
  podSelector:
    matchLabels:
      run: frontend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              run: backend
      ports:
        - protocol: TCP
          port: 80
```

Egress from the backend pod was also explicitly allowed to the frontend pod.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-egress-to-frontend
  namespace: security-lab
spec:
  podSelector:
    matchLabels:
      run: backend
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              run: frontend
      ports:
        - protocol: TCP
          port: 80
```

Final behaviour:

| Flow | Result |
|------|------|
| `backend → frontend` | Allowed |
| `attacker → frontend` | Denied |

This validated least-privilege pod communication using explicit ingress and egress rules.

## RBAC Model

Kubernetes RBAC was implemented to validate least-privilege API access.

Three access patterns were tested:

| Identity | Scope | Purpose |
|------|------|------|
| `readonly-user` | Cluster-wide read-only | Validate read-only access |
| `demo-admin` | Namespace-scoped admin | Validate admin rights limited to one namespace |
| `demo-deployer` | Custom namespace role | Validate minimal deployer permissions |

### Read-Only Identity

A read-only ServiceAccount was created in `rbac-lab` and bound to the built-in `view` ClusterRole.

Validated behaviour:

| Action | Result |
|------|------|
| Get pods across namespaces | Allowed |
| Delete pods | Denied |
| Create deployments | Denied |

This confirmed that the identity could inspect resources without modifying them.

### Namespace-Scoped Admin

A namespace-scoped admin identity was created for the `demo` namespace.

The ServiceAccount was bound to the built-in `admin` ClusterRole using a RoleBinding in `demo`.

Validated behaviour:

| Action | Namespace | Result |
|------|------|------|
| Create deployments | `demo` | Allowed |
| Create deployments | `security-lab` | Denied |
| Create cluster roles | cluster-wide | Denied |

This confirmed that admin access was limited to the intended namespace.

### Custom Least-Privilege Deployer Role

A custom deployer role was created for the `demo` namespace.

The role allows deployment-related operations but denies access to secrets, RBAC modification, and cluster-wide resources.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: demo-deployer-role
  namespace: demo
rules:
  - apiGroups: ["", "apps"]
    resources:
      - pods
      - services
      - configmaps
      - deployments
      - replicasets
    verbs:
      - get
      - list
      - watch
      - create
      - update
      - patch

  - apiGroups: [""]
    resources:
      - pods/log
    verbs:
      - get
```

Validated behaviour:

| Action | Result |
|------|------|
| Create deployments in `demo` | Allowed |
| Get pods in `demo` | Allowed |
| Get pod logs in `demo` | Allowed |
| Get secrets in `demo` | Denied |
| Create role bindings in `demo` | Denied |
| Create deployments in `security-lab` | Denied |

This confirmed that the custom role provides practical deployment permissions without granting unnecessary sensitive or administrative access.

## Validation

Validation was performed using positive and negative tests.

### NetworkPolicy Validation

| Test | Result |
|------|------|
| Baseline `backend → frontend` before policies | Allowed |
| Traffic after default deny | Blocked |
| DNS after default deny | Blocked |
| DNS after explicit allow policy | Restored |
| `backend → frontend` after allow policies | Allowed |
| `attacker → frontend` | Denied |

### RBAC Validation

RBAC permissions were validated using:

```bash
kubectl auth can-i
```

Examples:

```bash
kubectl auth can-i get pods \
  --as=system:serviceaccount:rbac-lab:readonly-user \
  -A
```

```bash
kubectl auth can-i delete pods \
  --as=system:serviceaccount:rbac-lab:readonly-user \
  -A
```

```bash
kubectl auth can-i create deployments \
  --as=system:serviceaccount:demo:demo-deployer \
  -n demo
```

```bash
kubectl auth can-i get secrets \
  --as=system:serviceaccount:demo:demo-deployer \
  -n demo
```

Final RBAC validation:

| Identity | Expected Access | Result |
|------|------|------|
| `readonly-user` | Read-only access | Enforced |
| `demo-admin` | Admin only in `demo` | Enforced |
| `demo-deployer` | Custom deployer permissions only | Enforced |
| Secrets access | Denied where not required | Enforced |
| RBAC modification | Denied where not required | Enforced |
| Cross-namespace access | Denied where not required | Enforced |

## Lessons Learned

Key takeaways from the Kubernetes security work included:

- NetworkPolicies are label-driven, not IP-driven
- label mismatches can cause policies to fail silently from an operator perspective
- `kubectl run` creates default labels such as `run=<name>`
- `kubectl get pods --show-labels` is essential when debugging NetworkPolicies
- default deny policies can also block DNS unless DNS egress is explicitly allowed
- both ingress and egress policies may be required for clear least-privilege communication
- RBAC should be validated with both allowed and denied `kubectl auth can-i` checks
- namespace-scoped RoleBindings are useful for limiting administrative access
- custom Roles provide better least-privilege control than broad built-in roles
- sensitive resources such as secrets and RBAC objects should not be granted unless explicitly required

The final Kubernetes security model combines default-deny networking with least-privilege API authorisation.
