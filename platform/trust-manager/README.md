# trust-manager

Distributes the corporate TLS-inspection proxy CA (`manifests/corporate-proxy-ca.yaml`)
cluster-wide as a `ConfigMap` (see `manifests/bundle.yaml`), so any pod's
outbound HTTPS traffic (git clone, helm/OCI pulls, etc.) that passes through
the firewall's TLS-inspecting proxy is trusted — without needing per-host,
per-namespace copies of the CA like `platform/argo-cd/values.yaml` used to
hardcode under `configs.tls.certificates`.

See `platform/cert-manager/README.md` for why this is bootstrapped manually,
before ArgoCD, instead of being installed as a GitOps-managed Application.

## Prerequisite

cert-manager must already be installed — run through
`platform/cert-manager/README.md` first. (trust-manager has no runtime
dependency on cert-manager itself; it's only used here to issue trust-manager's
own webhook certificate.)

## Manual bootstrap steps

```bash
# Install trust-manager from the OCI registry, matching cert-manager's
# installation source (https://cert-manager.io/docs/trust/trust-manager/installation/).
# The "trust namespace" defaults to "cert-manager", which is where
# corporate-proxy-ca lives, so no extra RBAC/config is needed to read it.
# No --version pin: always take the latest release for a fresh bootstrap.
helm upgrade trust-manager oci://quay.io/jetstack/charts/trust-manager \
  --install \
  --namespace cert-manager \
  --wait

# Apply the corporate proxy CA source ConfigMap, then the Bundle that
# assembles + distributes the trust bundle
kubectl apply -f platform/trust-manager/manifests/corporate-proxy-ca.yaml
kubectl apply -f platform/trust-manager/manifests/bundle.yaml
```

Once the GitOps Application for trust-manager is added under
`platform/applications/`, pin its `targetRevision` to whatever version ends
up installed here, so the manually bootstrapped release and the
ArgoCD-managed one stay in sync going forward.

Once this and `platform/cert-manager` are both bootstrapped, continue with
the ArgoCD install steps in the top-level README.

## Verifying

```bash
kubectl get bundle aips-trust-bundle
# Confirm the ConfigMap landed in a sample of namespaces
kubectl get configmap aips-trust-bundle -n cert-manager -o jsonpath='{.data.ca-bundle\.crt}' | head -5
kubectl get configmap aips-trust-bundle -n kube-system -o name
```
