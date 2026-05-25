# kubeadm Configuration Notes

This directory documents the kubeadm-based Kubernetes deployment model used in the lab.

The cluster was built manually with kubeadm on Debian virtual machines using containerd as the container runtime.

## Cluster Layout

| Role | Node | Address |
|------|------|------|
| Control plane | `k8s-master` | `10.10.10.10` |
| Worker | `worker1` | `10.10.10.11` |
| Worker | `worker2` | `10.10.10.12` |
| Worker | `worker3` | `10.10.10.13` |

## Runtime Configuration

containerd is used as the Kubernetes container runtime.

The runtime was configured with `SystemdCgroup` enabled to align with the Kubernetes node configuration.

## Networking

Calico is used as the Kubernetes CNI plugin.

During deployment, the CNI plugin path was validated and corrected so that Kubernetes could locate the required CNI binaries.

## Notes

This directory documents the deployment model rather than providing a full automated cluster bootstrap.

The detailed Kubernetes platform design is documented in:

- [`docs/kubernetes.md`](../../docs/kubernetes.md)
- [`docs/networking.md`](../../docs/networking.md)
- [`docs/troubleshooting.md`](../../docs/troubleshooting.md)
