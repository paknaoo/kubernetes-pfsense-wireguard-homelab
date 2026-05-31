# Troubleshooting & Lessons Learned

## Overview

The project involved diagnosing and resolving issues across networking, Kubernetes deployment, container networking, and observability components.

The troubleshooting process focused on identifying root causes, validating fixes, and restoring expected platform behaviour.

## Contents

- [DHCP & Addressing Issues](#dhcp--addressing-issues)
- [Calico Deployment Issues](#calico-deployment-issues)
- [CNI Path Mismatch](#cni-path-mismatch)
- [metrics-server Compatibility](#metrics-server-compatibility)
- [WireGuard Full-Tunnel Internet Egress](#wireguard-full-tunnel-internet-egress)
- [NetworkPolicy Label Matching](#networkpolicy-label-matching)
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

## WireGuard Full-Tunnel Internet Egress

### Problem

WireGuard connectivity to internal lab resources worked, but Internet access from `mgmt01` through the VPN tunnel failed.

The goal was to route `mgmt01` Internet traffic through WireGuard, pfSense, and the OPT1 Internet uplink.

Expected flow:

```text
mgmt01 → WireGuard → pfSense → OPT1 → Internet
```

### Cause

Several issues contributed to the failure:

- missing required Linux tools on `mgmt01`
- `wg-quick` unable to locate required binaries because root was entered with `su` instead of `su -`
- missing outbound NAT rule for the WireGuard subnet
- incorrect assumption that WireGuard full-tunnel routing would appear in the main routing table

Using `su` without `-` did not load the full root login environment, which meant paths such as `/usr/sbin` were missing from `PATH`.

As a result, tools required by `wg-quick` appeared unavailable even when the relevant packages were already installed.

Required tools included:

```text
wg
sysctl
iptables-restore
```

WireGuard full-tunnel routing used policy routing through table `51820`, rather than a standard `default dev wg0` route in the main routing table.

### Resolution

The required packages were verified or installed on `mgmt01`:

```bash
sudo apt install -y wireguard-tools iptables procps
```

The root environment issue was resolved by using a proper root login shell:

```bash
su -
```

instead of:

```bash
su
```

Using `su -` loads the root login environment, including system paths such as:

```text
/usr/sbin
/sbin
```

This allowed tools such as `sysctl` and `iptables-restore` to be found correctly by `wg-quick`.

pfSense was configured with Hybrid Outbound NAT and a NAT rule translating the WireGuard subnet through the OPT1 interface.

| Interface | Source | Destination | Translation |
|------|------|------|------|
| OPT1 | `10.20.20.0/24` | any | OPT1 address |

The pfSense default gateway was set to:

```text
OPT1_DHCP
```

### Validation

The tunnel and routing behaviour were validated using:

```bash
sudo wg
ip route show table 51820
ip rule
curl -I https://google.com
curl ifconfig.me
```

Expected routing behaviour:

```text
table 51820:
default dev wg0
```

### Outcome

Successful restoration of:

- WireGuard full-tunnel routing
- Internet access through pfSense OPT1
- outbound NAT for the WireGuard subnet
- continued restricted access to internal Kubernetes resources

## NetworkPolicy Label Matching

### Problem

NetworkPolicy rules were applied successfully, but expected pod communication still timed out.

The policy YAML was valid, but traffic did not match the intended allow rules.

### Cause

NetworkPolicies select pods using labels, not pod names, service names, or IP addresses.

The initial policy used labels such as:

```yaml
app: frontend
```

However, the test pods created with `kubectl run` used default labels:

```text
run=frontend
run=backend
```

As a result, the policy did not match the intended pods.

### Resolution

Pod labels were inspected with:

```bash
kubectl get pods -n security-lab --show-labels
```

The NetworkPolicies were updated to match the actual labels:

```text
run=frontend
run=backend
```

After correcting the selectors, the intended behaviour was validated:

| Flow | Result |
|------|------|
| `backend → frontend` | Allowed |
| `attacker → frontend` | Denied |

### Outcome

The issue reinforced that NetworkPolicies are label-driven and that label validation should be one of the first troubleshooting steps when policy behaviour does not match expectations.

## Lessons Learned

Key takeaways from the project included:

- predictable infrastructure addressing is critical for stable Kubernetes operation
- Kubernetes networking issues often propagate into dependent platform components
- self-managed Kubernetes environments require careful integration between networking, runtime, firewalling, and platform services
- using `su` instead of `su -` can result in an incomplete root environment, causing administrative tools in `/usr/sbin` to appear unavailable
- WireGuard full-tunnel routing may use policy routing tables rather than the main routing table
- outbound NAT is required when routing VPN client traffic to the Internet through a dedicated pfSense uplink
- validating root causes is more effective than applying configuration changes blindly
- operational testing is essential after every infrastructure change
- NetworkPolicies are label-driven, so pod labels should be verified before troubleshooting policy logic
- `kubectl run` creates default `run=<name>` labels, which may differ from expected `app=<name>` labels
