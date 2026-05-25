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

## Calico Deployment Issues

### Problem

Cluster networking components failed to initialise correctly during early Kubernetes platform bring-up.

Affected symptoms included incomplete Calico deployment and Kubernetes workloads remaining unavailable.

### Cause

The Calico installation was missing the required Kubernetes custom resource definitions and supporting resources.

As a result, the networking layer was not fully established.

### Resolution

The missing Calico resources were deployed and cluster networking components were revalidated.

Following remediation, Calico initialised successfully and Kubernetes networking behaviour normalised.

### Outcome

Successful restoration of:

- Calico pod networking
- Kubernetes workload scheduling
- cluster networking functionality

## CNI Path Mismatch

### Problem

Several Kubernetes workloads, including CoreDNS, remained stuck in a `ContainerCreating` state after cluster deployment.

Pod networking was not functioning correctly across the environment.

### Cause

A mismatch existed between the expected Kubernetes CNI plugin location and the installed plugin path.

Expected location:

```text
/usr/lib/cni
```

Installed plugin location:

```text
/opt/cni/bin
```

This prevented Kubernetes from locating the required CNI binaries.

### Resolution

The issue was resolved by aligning the expected CNI path with the installed plugin location through a filesystem symlink.

Following remediation, Kubernetes networking components initialised successfully.

### Outcome

Successful restoration of:

- pod networking
- CoreDNS startup
- normal Kubernetes workload operation

## metrics-server Compatibility

### Problem

metrics-server was unable to collect resource metrics from cluster nodes during deployment.

As a result, standard Kubernetes metrics queries were unavailable.

### Cause

Compatibility differences between metrics-server and kubelet TLS behaviour prevented successful metrics collection within the lab environment.

### Resolution

metrics-server configuration was adjusted to accommodate the kubelet TLS behaviour used by the cluster.

Following the configuration change, metrics collection resumed successfully.

### Outcome

Validated functionality:

```bash
kubectl top nodes
kubectl top pods -A
```

## Lessons Learned

Key takeaways from the project included:

- predictable infrastructure addressing is critical for stable Kubernetes operation
- Kubernetes networking issues often propagate into dependent platform components
- validating root causes is more effective than applying configuration changes blindly
- self-managed Kubernetes environments require careful integration between networking, runtime, and platform services
- operational testing is essential after every infrastructure change
