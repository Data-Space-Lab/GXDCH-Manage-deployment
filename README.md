# GXDCH Manage deployment

This repository contains the Kubernetes and ArgoCD manifests for GXDCH Manage.
The application image is built from `GXDCH-Manage-source` and published as
`ghcr.io/data-space-lab/gxdch-manage`.

Before syncing the ArgoCD application, create `gxdch-manage-secret` in the
`gxdch-manage` namespace from `secret.example.yaml`. The `ISSUER_ADMIN_API_KEY`
must be the GXDCH issuer super-user key. The secret is intentionally excluded
from Kustomize so credentials are never committed to Git.

The GHCR package is currently private. Create or update the pull secret before
syncing:

```bash
kubectl -n gxdch-manage create secret docker-registry ghcr-pull \
  --docker-server=ghcr.io \
  --docker-username=YOUR_GITHUB_USER \
  --docker-password=YOUR_GITHUB_TOKEN \
  --dry-run=client -o yaml | kubectl apply -f -
```

The token needs permission to read packages (`read:packages`).

The UI stores connector onboarding metadata in the `gxdch-manage-data` PVC.
