# Kubernetes + pfSense + WireGuard Homelab

## Overview

A security-focused homelab built to explore Kubernetes platform engineering, network segmentation, and secure remote administration.

The environment combines a kubeadm-based Kubernetes cluster, pfSense firewalling, WireGuard VPN access, Envoy Gateway, MetalLB, and a least-privilege access model running on VMware Workstation.

The project is designed as a practical platform for deploying, validating, and troubleshooting production-style infrastructure components.

## Technology Stack

### Infrastructure

- VMware Workstation
- pfSense
- Debian Linux

### Kubernetes Platform

- Kubernetes (kubeadm)
- containerd
- Calico CNI
- metrics-server
- local-path-provisioner

### Networking & Access

- WireGuard VPN
- Envoy Gateway
- Gateway API
- MetalLB

## Architecture

The lab is built around a segmented network design using pfSense as the routing, firewall, and VPN boundary.

A kubeadm-based Kubernetes cluster runs inside the LAN network, while remote administration is performed through a restricted WireGuard VPN path from a dedicated management workstation.

```mermaid
flowchart LR

    OUTSIDE["OUTSIDE Network
    192.168.50.0/24"]

    MGMT["mgmt01
    192.168.50.10
    Management Workstation"]

    PFSENSE["pfSense
    WAN: 192.168.50.254
    LAN: 10.10.10.254
    WG: 10.20.20.1"]

    subgraph LAN["Kubernetes LAN — 10.10.10.0/24"]

        MASTER["k8s-master
        10.10.10.10"]

        W1["worker1
        10.10.10.11"]

        W2["worker2
        10.10.10.12"]

        W3["worker3
        10.10.10.13"]

        INGRESS["Envoy Gateway / MetalLB
        10.10.10.50"]
    end

    WG["WireGuard VPN
    10.20.20.0/24"]

    OUTSIDE --> MGMT
    MGMT --> WG
    WG --> PFSENSE
    PFSENSE --> LAN
```
