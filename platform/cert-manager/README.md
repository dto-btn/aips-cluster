# cert-manager

Issues and renews public TLS certificates cluster-wide via ACME/Azure DNS
(`manifests/cluster-issuer.yaml`). This is a prerequisite for trust-manager
(see `../trust-manager/`), which is what actually solves the corporate
firewall CA-trust problem — cert-manager itself has no internal-CA/root-CA
role in this repo.

## Why this is installed manually, before ArgoCD

cert-manager + trust-manager are meant to be GitOps-managed by ArgoCD long
term (see `sourceRepos` in `platform/argo-cd/projects/aips-platform.yaml`,
which already whitelists `oci://quay.io/jetstack/charts/cert-manager`). But
ArgoCD's own workloads need the corporate proxy CA trusted (via trust-manager)
from the moment they start, so they can reach GitHub/quay.io/etc. through the
TLS-inspecting firewall. That's a chicken-and-egg problem: ArgoCD can't be
relied on to install the thing that needs to exist before ArgoCD starts.

The fix is a one-time manual bootstrap: install cert-manager + trust-manager
*before* the `helm install argocd ...` step in the top-level README. Once
ArgoCD is up, it will reconcile these same charts/manifests going forward
(adopting them, not recreating them) via a future
`platform/applications/cert-manager.yaml` / `trust-manager.yaml` Application —
safe because the Helm release name/namespace and manifest content will match
exactly what was applied manually.

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

# 3. Apply the existing public ACME issuer
kubectl apply -f platform/cert-manager/manifests/cluster-issuer.yaml
```

Once the GitOps Application for cert-manager is added under
`platform/applications/`, pin its `targetRevision` to whatever version ends
up installed here, so the manually bootstrapped release and the
ArgoCD-managed one stay in sync going forward.

Next: [`platform/trust-manager/README.md`](../trust-manager/README.md) to
actually distribute the corporate proxy CA cluster-wide, before continuing
with the ArgoCD bootstrap in the top-level README.

## Verifying

```bash
kubectl get clusterissuers
kubectl get pods -n cert-manager
```
