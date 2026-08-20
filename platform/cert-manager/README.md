# cert-manager

Issues and renews TLS certificates cluster-wide: public certs via ACME/Azure DNS
(`manifests/cluster-issuer.yaml`) and internal certs via a self-signed root CA
that this repo bootstraps (`selfsigned-bootstrap-issuer.yaml` →
`root-ca-certificate.yaml` → `ca-cluster-issuer.yaml`).

## Why this is installed manually, before ArgoCD

ArgoCD's own workloads (and, later, the products it deploys) need to be able to
trust certificates issued by our internal CA from the moment they start. But
cert-manager + trust-manager are themselves meant to be GitOps-managed by
ArgoCD (see `sourceRepos` in `platform/argo-cd/projects/aips-platform.yaml`,
which already whitelists `oci://quay.io/jetstack/charts/cert-manager`). That's
a chicken-and-egg problem: ArgoCD can't be relied on to install the thing that
needs to exist before ArgoCD starts.

The fix is a one-time manual bootstrap: install cert-manager + trust-manager,
mint the root CA, and let trust-manager distribute it *before* the `helm
install argocd ...` step in the top-level README. Once ArgoCD is up, it will
reconcile these same charts/manifests going forward (adopting them, not
recreating them) via a future `platform/applications/cert-manager.yaml` /
`trust-manager.yaml` Application — safe because the Helm release
name/namespace and manifest content will match exactly what was applied
manually.

## Manual bootstrap steps

```bash
# 1. Namespace
kubectl create namespace cert-manager

# 2. Install cert-manager from the OCI registry (recommended over the
#    legacy https://charts.jetstack.io repo, per
#    https://cert-manager.io/docs/installation/helm/). Installs its own CRDs.
#    No --version pin: always take the latest release for a fresh bootstrap.
helm upgrade cert-manager oci://quay.io/jetstack/charts/cert-manager \
  --install \
  --namespace cert-manager \
  --set crds.enabled=true \
  --wait

# 3. Bootstrap the self-signed issuer + internal root CA
kubectl apply -f platform/cert-manager/manifests/selfsigned-bootstrap-issuer.yaml
kubectl apply -f platform/cert-manager/manifests/root-ca-certificate.yaml

# wait for the root CA Certificate to become Ready before continuing
kubectl wait --for=condition=Ready certificate/aips-root-ca -n cert-manager --timeout=120s

# 4. Real cluster-wide issuer backed by that root CA, and the existing
#    public ACME issuer
kubectl apply -f platform/cert-manager/manifests/ca-cluster-issuer.yaml
kubectl apply -f platform/cert-manager/manifests/cluster-issuer.yaml
```

Once the GitOps Application for cert-manager is added under
`platform/applications/`, pin its `targetRevision` to whatever version ends
up installed here, so the manually bootstrapped release and the
ArgoCD-managed one stay in sync going forward.

Next: [`platform/trust-manager/README.md`](../trust-manager/README.md) to
distribute `aips-root-ca-secret` cluster-wide, before continuing with the
ArgoCD bootstrap in the top-level README.

## Verifying

```bash
kubectl get clusterissuers
kubectl get certificate aips-root-ca -n cert-manager
kubectl get secret aips-root-ca-secret -n cert-manager
```
