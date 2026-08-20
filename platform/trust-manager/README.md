# trust-manager

Distributes the AIPS internal root CA (minted in
`platform/cert-manager/manifests/root-ca-certificate.yaml`) cluster-wide as a
`ConfigMap` (see `manifests/bundle.yaml`), so any pod can trust certificates
issued by `aips-internal-ca-issuer` without per-namespace copies of the CA.

See `platform/cert-manager/README.md` for why this is bootstrapped manually,
before ArgoCD, instead of being installed as a GitOps-managed Application.

## Prerequisite

cert-manager must already be installed and the root CA
(`aips-root-ca-secret` in the `cert-manager` namespace) must already exist —
run through `platform/cert-manager/README.md` first.

## Manual bootstrap steps

```bash
# Install trust-manager from the OCI registry, matching cert-manager's
# installation source (https://cert-manager.io/docs/trust/trust-manager/installation/).
# The "trust namespace" defaults to "cert-manager", which is where
# aips-root-ca-secret lives, so no extra RBAC/config is needed to read it.
# No --version pin: always take the latest release for a fresh bootstrap.
helm upgrade trust-manager oci://quay.io/jetstack/charts/trust-manager \
  --install \
  --namespace cert-manager \
  --wait

# Apply the Bundle that assembles + distributes the trust bundle
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
