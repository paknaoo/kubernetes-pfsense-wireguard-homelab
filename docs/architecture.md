# Architecture

## Overview

This document describes the high-level architecture of the Kubernetes, pfSense, and WireGuard homelab.

The environment is designed around segmented networking, controlled administrative access, and a self-managed Kubernetes platform running on VMware Workstation.

pfSense provides the core network boundary, WireGuard VPN termination, controlled Internet egress through OPT1, and internal Kubernetes service exposure through MetalLB and Envoy Gateway.

## Contents

- [High-Level Architecture](#high-level-architecture)
- [Infrastructure Components](#infrastructure-components)
- [Network Architecture](#network-architecture)
- [Kubernetes Architecture](#kubernetes-architecture)
- [Administration Flow](#administration-flow)
- [SSH Administration Flow](#ssh-administration-flow)
- [Architecture Validation](#architecture-validation)

## High-Level Architecture

The lab is built around pfSense as the central routing, firewall, VPN, and egress control component.

Kubernetes nodes run inside the LAN segment, while the management workstation connects through WireGuard for restricted administrative access and full-tunnel Internet egress through the OPT1 uplink.

```mermaid
flowchart LR

    subgraph OUTSIDE["OUTSIDE Network - 192.168.50.0/24"]
        MGMT["mgmt01<br>Management Workstation<br>192.168.50.10"]
    end

    PFSENSE["pfSense<br>WAN: 192.168.50.254<br>LAN: 10.10.10.254<br>WG: 10.20.20.1<br>OPT1: DHCP Internet uplink"]

    INTERNET["Internet"]

    subgraph WG["WireGuard Network - 10.20.20.0/24"]
        VPNCLIENT["mgmt01 VPN<br>10.20.20.2<br>Full tunnel"]
    end

    subgraph LAN["LAN Network - 10.10.10.0/24"]
        MASTER["k8s-master<br>10.10.10.10"]
        W1["worker1<br>10.10.10.11"]
        W2["worker2<br>10.10.10.12"]
        W3["worker3<br>10.10.10.13"]
        INGRESS["Envoy Gateway / MetalLB<br>10.10.10.50"]
    end

    MGMT --> PFSENSE
    MGMT --> VPNCLIENT
    VPNCLIENT --> PFSENSE

    PFSENSE --> MASTER
    PFSENSE --> INGRESS
    PFSENSE --> INTERNET

    MASTER --> W1
    MASTER --> W2
    MASTER --> W3
```

## Infrastructure Components

The lab runs on VMware Workstation and is composed of dedicated virtual machines for firewalling, management, and Kubernetes workloads.

| Component | Role |
|------|------|
| VMware Workstation | Local hypervisor platform |
| pfSense | Router, firewall, DHCP server, WireGuard VPN endpoint, and controlled Internet egress gateway |
| OPT1 uplink | pfSense Internet egress interface used for WireGuard full-tunnel traffic |
| mgmt01 | External management workstation and VPN client |
| k8s-master | Kubernetes control plane node |
| worker1 | Kubernetes worker node |
| worker2 | Kubernetes worker node |
| worker3 | Kubernetes worker node |

This separation keeps network control, administration, and Kubernetes workloads clearly defined across the environment.

## Network Architecture

The environment uses three primary network segments and a dedicated OPT1 Internet uplink.

| Network / Interface | Address Range | Purpose |
|------|------|------|
| OUTSIDE | `192.168.50.0/24` | External / hypervisor-facing network |
| LAN | `10.10.10.0/24` | Kubernetes cluster network |
| WG | `10.20.20.0/24` | WireGuard VPN network |
| OPT1 | DHCP | Internet uplink for controlled VPN egress |

pfSense routes traffic between these segments and enforces firewall policy at the network boundaries.

Kubernetes services are exposed inside the LAN network using MetalLB, with the current Envoy Gateway ingress address allocated as `10.10.10.50`.

WireGuard is configured in full-tunnel mode for `mgmt01`, allowing Internet traffic to egress through pfSense via the OPT1 uplink while keeping internal LAN access restricted to approved targets.

More detailed routing, service exposure, and traffic flow notes are covered in [Networking Design](networking.md).

## Kubernetes Architecture

The Kubernetes platform is deployed with `kubeadm` on Debian virtual machines using `containerd` as the container runtime.

The cluster consists of one control plane node and three worker nodes.

| Layer | Component |
|------|------|
| Control plane | `k8s-master` |
| Worker nodes | `worker1`, `worker2`, `worker3` |
| Container runtime | `containerd` |
| Pod networking | Calico |
| DNS | CoreDNS |
| Metrics | metrics-server |
| Storage | local-path-provisioner |
| Service exposure | MetalLB |
| Ingress / Gateway | Envoy Gateway + Gateway API |

The platform supports core Kubernetes workflows including workload scheduling, pod networking, persistent storage, resource metrics, and HTTP ingress routing.

More detailed Kubernetes platform notes are covered in [Kubernetes Platform](kubernetes.md).

## Administration Flow

Remote administration is performed from `mgmt01` through a WireGuard VPN tunnel terminated on pfSense.

The VPN client uses full-tunnel routing for Internet egress through pfSense OPT1, while access to internal LAN resources remains restricted to approved management targets.

```mermaid
flowchart TB

    subgraph OUTSIDE["OUTSIDE Network"]
        MGMT["mgmt01<br>Management Workstation"]
    end

    subgraph VPN["WireGuard VPN"]
        WG["VPN Tunnel<br>10.20.20.0/24<br>Full tunnel"]
    end

    subgraph FIREWALL["pfSense Boundary"]
        PFSENSE["pfSense<br>Firewall / Router / VPN"]
    end

    subgraph ALLOWED["Approved Internal VPN Targets"]
        GUI["pfSense GUI<br>10.10.10.254"]
        MASTER["k8s-master<br>10.10.10.10<br>Control Plane / Admin Entry Point"]
        INGRESS["Envoy Gateway<br>10.10.10.50"]
    end

    subgraph WORKERS["Worker Nodes"]
        W1["worker1<br>10.10.10.11"]
        W2["worker2<br>10.10.10.12"]
        W3["worker3<br>10.10.10.13"]
    end

    OPT1["OPT1<br>Internet Uplink"]
    INTERNET["Internet"]
    BLOCKED["Remaining LAN<br>Blocked from VPN"]

    MGMT --> WG
    WG --> PFSENSE

    PFSENSE --> GUI
    PFSENSE --> MASTER
    PFSENSE --> INGRESS

    MASTER --> W1
    MASTER --> W2
    MASTER --> W3

    PFSENSE --> OPT1
    OPT1 --> INTERNET

    PFSENSE -. direct VPN access blocked .-> WORKERS
    PFSENSE -. blocked .-> BLOCKED
```
This model allows remote administration of the cluster while keeping worker nodes and the wider LAN inaccessible directly from the VPN network.

Worker administration is performed through the Kubernetes control plane node, preserving the restricted access model while still allowing practical cluster management.

Internet traffic from `mgmt01` is routed through the WireGuard tunnel and exits through pfSense using the OPT1 uplink.

More detailed firewall and VPN notes are covered in [Access Control](access-control.md).

## SSH Administration Flow

SSH administration follows a VPN + jump host model.

The management workstation connects directly only to the Kubernetes control plane node. Worker nodes are administered through SSH ProxyJump using `k8s-master` as the jump host.

```mermaid
flowchart TB

    MGMT["mgmt01<br>Management Workstation"]

    WG["WireGuard VPN"]

    MASTER["k8s-master<br>10.10.10.10<br>SSH Jump Host"]

    subgraph WORKERS["Worker Nodes"]
        W1["worker1<br>10.10.10.11"]
        W2["worker2<br>10.10.10.12"]
        W3["worker3<br>10.10.10.13"]
    end

    BLOCKED["Direct SSH to workers<br>Blocked"]

    MGMT --> WG
    WG --> MASTER

    MASTER --> W1
    MASTER --> W2
    MASTER --> W3

    WG -. blocked .-> BLOCKED
```

This model supports practical worker administration while avoiding direct SSH exposure of worker nodes to the VPN client.

More detailed SSH hardening notes are covered in [SSH Hardening](ssh-hardening.md).

## Architecture Validation

The architecture has been validated across infrastructure, networking, Kubernetes platform services, ingress routing, and secure remote administration.

| Area | Validation |
|------|------|
| Network segmentation | OUTSIDE, LAN, and WG networks separated through pfSense |
| Routing | pfSense routing operational between required segments |
| Kubernetes cluster | All nodes reporting `Ready` |
| Pod networking | Calico operational |
| Service exposure | MetalLB assigning ingress address from LAN pool |
| Gateway routing | Envoy Gateway routing traffic through Gateway API resources |
| VPN administration | WireGuard tunnel established from `mgmt01` |
| Internet egress | WireGuard full-tunnel traffic routed through pfSense OPT1 |
| Outbound NAT | WireGuard subnet translated through the OPT1 interface |
| Access control | VPN access restricted to approved management targets |
| SSH access model | Worker administration performed through `k8s-master` as jump host |
| Direct worker SSH | Direct SSH from VPN client to worker nodes blocked |
| SSH hardening | Key-based SSH, root login disabled, password login disabled, and Fail2ban active |

The validated architecture provides a working self-managed Kubernetes environment with controlled service exposure and restricted administrative access.
