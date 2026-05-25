# Networking Design

## Overview

The lab networking model is built around segmented network boundaries, controlled routing, and service exposure suitable for a self-managed Kubernetes environment.

pfSense provides routing, firewalling, DHCP services, and VPN termination, while Kubernetes networking is handled through Calico, MetalLB, and Envoy Gateway.

## Contents

- [Overview](#overview)
- [Network Segments](#network-segments)
- [Routing Model](#routing-model)
- [Kubernetes Networking](#kubernetes-networking)
- [MetalLB Service Exposure](#metallb-service-exposure)
- [Gateway & Traffic Flow](#gateway--traffic-flow)
- [Network Validation](#network-validation)
