# Storage Manifests

This directory contains a simple persistent storage validation workload for the Kubernetes lab.

The test uses `local-path-provisioner` to create a PVC-backed workload and verify that data can be written to mounted persistent storage.

## Files

| File | Purpose |
|------|------|
| `pvc-test.yaml` | Creates a test PVC and Pod using the `local-path` StorageClass |

## Validation

After applying the manifest, storage can be validated with:

```bash
kubectl apply -f pvc-test.yaml
kubectl logs local-path-test-pod
```

Expected output:

```text
persistent storage test
```

The Pod remains running temporarily to allow inspection of the mounted volume.
