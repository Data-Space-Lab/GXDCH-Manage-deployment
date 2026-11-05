# GXDCH Manage deployment

This repository contains the Helm chart and ArgoCD configuration for GXDCH Manage.
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

### Provision through Argo CD

The Helm chart can create `ghcr-pull` during an Argo sync. In the Argo CD UI,
open the `gxdch-manage` application, choose **Details**, edit **Parameters**,
and add:

```text
registry.pullSecret.create=true
registry.pullSecret.username=<github-username>
registry.pullSecret.password=<github-token>
```

The Secret is created in the application namespace and is already referenced
by the GXDCH Manage Deployment. This keeps the token out of Git, but the value
is stored in the Argo Application override. Use this only for trusted Argo
administrators; for production, provision the Secret externally or with a
secret operator and keep `registry.pullSecret.create=false`.

The UI stores connector onboarding metadata in the `gxdch-manage-data` PVC.

During membership issuance, GXDCH Manage calls the connector's authenticated
`/mgmt/dcp/credential-requests` endpoint. The connector signs the DCP
`CredentialRequestMessage` with the private key corresponding to its published
DID document. Set `CONNECTOR_MANAGEMENT_AUTHORIZATION` in
`deploy/configmap.yaml` to the management authorization accepted by the
connector. Do not put a private signing key in this repository.

The `route.yaml` and `referencegrant.yaml` files are kept separately because
the vcluster does not provide Gateway API CRDs. Apply those two files in the
host cluster that owns the `infra` Gateway, or configure an equivalent route
there.

## Helm profiles

The chart has a GXDCH default profile and an OULUDS profile:

```bash
helm lint chart/gxdch-manage
helm template gxdch-manage chart/gxdch-manage \
  -f chart/gxdch-manage/values-ouluds.yaml
```

Secrets and the existing SQLite PVC are intentionally not rendered by the
chart. The OULUDS ArgoCD Application should use the chart with
`values-ouluds.yaml`.
