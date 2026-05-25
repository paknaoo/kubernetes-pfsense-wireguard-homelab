# Envoy Gateway Manifests

This directory contains the Gateway API resources used to expose the test application through Envoy Gateway.

The setup demonstrates HTTP ingress routing inside the Kubernetes lab using Gateway API resources and a MetalLB-assigned ingress address.

## Components

| Resource | Purpose |
|------|------|
| `GatewayClass` | Defines Envoy Gateway as the Gateway API controller |
| `Gateway` | Creates an HTTP listener for `app.lab.local` |
| `HTTPRoute` | Routes HTTP traffic to the backend service |
| `httpbin` | Test backend application used for ingress validation |

## Files

| File | Purpose |
|------|------|
| `gatewayclass.yaml` | GatewayClass definition |
| `gateway.yaml` | Gateway listener configuration |
| `httproute.yaml` | HTTPRoute rule for `app.lab.local` |
| `httpbin.yaml` | Test backend Deployment and Service |

## Validation

Ingress routing was validated with:

```bash
curl -H "Host: app.lab.local" http://10.10.10.50/get
