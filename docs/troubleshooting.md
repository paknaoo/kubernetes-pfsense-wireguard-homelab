# Troubleshooting & Lessons Learned

## Overview

The project involved diagnosing and resolving issues across networking, Kubernetes deployment, container networking, and observability components.

The troubleshooting process focused on identifying root causes, validating fixes, and restoring expected platform behaviour.

## Contents

- [DHCP & Addressing Issues](#dhcp--addressing-issues)
- [Calico Deployment Issues](#calico-deployment-issues)
- [CNI Path Mismatch](#cni-path-mismatch)
- [metrics-server Compatibility](#metrics-server-compatibility)
- [Lessons Learned](#lessons-learned)

## DHCP & Addressing Issues

### Problem

Unexpected DHCP behaviour resulted in inconsistent addressing and duplicate IP assignment during early infrastructure deployment.

### Cause

Static DHCP mappings were configured before virtual machine deployment, which contributed to lease inconsistencies and unexpected address allocation behaviour.

### Resolution

The issue was resolved through DHCP lease cleanup, addressing verification, and rebuilding the affected mappings.

Following remediation, Kubernetes nodes received the expected static addresses and stable network behaviour was restored.

### Outcome

Predictable addressing was re-established for:

- `k8s-master`
- `worker1`
- `worker2`
- `worker3`

