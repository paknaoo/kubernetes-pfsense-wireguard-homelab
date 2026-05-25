# Networking Design

## Overview

The lab networking model is built around segmented network boundaries, controlled routing, and service exposure suitable for a self-managed Kubernetes environment.

pfSense provides routing, firewalling, DHCP services, and VPN termination, while Kubernetes networking is handled through Calico, MetalLB, and Envoy Gateway.

## Contents

- [Network Segments](#network-segments)
- [Routing Model](#routing-model)
- [Kubernetes Networking](#kubernetes-networking)
- [MetalLB Service Exposure](#metallb-service-exposure)
- [Gateway & Traffic Flow](#gateway--traffic-flow)
- [Network Validation](#network-validation)

## Network Segments

The environment uses three dedicated network segments with distinct operational roles.

| Network | Purpose | Address Range |
|------|------|------|
| OUTSIDE | External / hypervisor-facing network | `192.168.50.0/24` |
| LAN | Kubernetes cluster network | `10.10.10.0/24` |
| WG | WireGuard administrative network | `10.20.20.0/24` |

pfSense provides the network boundary between these segments and controls routing, firewall enforcement, and VPN connectivity.

```mermaid
flowchart LR

    OUTSIDE["OUTSIDE
    192.168.50.0/24"]

    PFSENSE["pfSense
    Router / Firewall / VPN"]

    LAN["LAN
    10.10.10.0/24"]

    WG["WireGuard
    10.20.20.0/24"]

    OUTSIDE --> PFSENSE
    WG --> PFSENSE
    PFSENSE --> LAN
```

## Routing Model

pfSense acts as the central routing point for the lab environment.

### Interface Configuration

| Interface | Address |
|------|------|
| WAN / OUTSIDE | `192.168.50.254` |
| LAN | `10.10.10.254` |
| WireGuard | `10.20.20.1` |

### DHCP & Static Addressing

LAN DHCP is provided by pfSense.

Static mappings are used for Kubernetes infrastructure nodes to maintain predictable addressing and stable cluster behaviour.

| Node | Address |
|------|------|
| k8s-master | `10.10.10.10` |
| worker1 | `10.10.10.11` |
| worker2 | `10.10.10.12` |
| worker3 | `10.10.10.13` |

### Administrative Traffic Path

The management workstation (`mgmt01`) operates from the OUTSIDE network (`192.168.50.10`).

Administrative traffic follows the path:

```text
mgmt01 → WireGuard VPN → pfSense → approved internal resources
```

This routing model allows secure remote administration while keeping internal Kubernetes services isolated from direct external access.

## Kubernetes Networking

Kubernetes pod networking is provided by Calico.

Calico enables pod-to-pod communication across the Debian cluster nodes and integrates with the kubeadm-based cluster deployment.

CoreDNS is running successfully and provides internal Kubernetes service discovery.

### Container Networking Interface

The cluster uses the standard Kubernetes CNI model for pod networking.

During deployment, a CNI plugin path mismatch was identified and resolved by aligning the expected CNI binary location with the installed plugin path.

This restored pod networking and allowed CoreDNS and other cluster workloads to start correctly.

## MetalLB Service Exposure

MetalLB provides `LoadBalancer` functionality for the self-managed Kubernetes environment.

A dedicated address pool is allocated from the LAN network for service exposure.

### Address Pool

| Component | Range |
|------|------|
| MetalLB IP Pool | `10.10.10.50 – 10.10.10.60` |

### Current Ingress Allocation

| Service | Address |
|------|------|
| Envoy Gateway ingress | `10.10.10.50` |

This approach enables Kubernetes services to be exposed within the lab environment without relying on a cloud provider load balancer.

## Gateway & Traffic Flow

Ingress traffic management is implemented using Envoy Gateway and the Kubernetes Gateway API.

The deployment includes:

- GatewayClass
- Gateway
- HTTPRoute
- `httpbin` backend service

Incoming requests are routed through the Envoy Gateway ingress address exposed by MetalLB.

```mermaid
flowchart LR

    CLIENT["Client Request"]

    INGRESS["Envoy Gateway
    10.10.10.50"]

    ROUTE["HTTPRoute"]

    APP["httpbin backend"]

    CLIENT --> INGRESS
    INGRESS --> ROUTE
    ROUTE --> APP
```

Validated request flow:

```bash
curl -H "Host: app.lab.local" http://10.10.10.50/get
```

Successful responses confirm correct integration between MetalLB, Envoy Gateway, Gateway API resources, and backend service routing.

