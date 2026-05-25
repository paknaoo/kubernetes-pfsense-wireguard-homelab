# metrics-server Notes

This directory documents the metrics-server configuration adjustment used in the Kubernetes lab.

metrics-server provides resource metrics for Kubernetes nodes and pods, enabling standard Kubernetes resource monitoring commands.

## Purpose

metrics-server was installed to provide basic observability for the cluster and to validate that node and workload metrics could be collected successfully.

Validated commands:

```bash
kubectl top nodes
kubectl top pods -A
```

## Lab Adjustment

During deployment, metrics-server required a kubelet TLS compatibility adjustment in the lab environment.

The deployment was configured with the following argument:

```text
--kubelet-insecure-tls
```

This allowed metrics-server to collect resource metrics from kubelets in the controlled lab setup.

## Validation

After applying the adjustment, metrics collection was validated successfully.

Expected behaviour:

- node resource metrics are available
- pod resource metrics are available across namespaces
- `kubectl top` commands return data successfully

Example validation commands:

```bash
kubectl top nodes
kubectl top pods -A
```

## Production Note

This adjustment is suitable for the controlled lab environment documented in this repository.

For production environments, kubelet certificate trust should be configured properly rather than bypassing TLS verification.
