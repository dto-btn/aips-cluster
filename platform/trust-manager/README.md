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
# Keep this version aligned with platform/applications/trust-manager.yaml.
helm upgrade trust-manager oci://quay.io/jetstack/charts/trust-manager \
  --install \
  --namespace cert-manager \
  --version 0.25.0 \
  --set defaultPackageImage.repository=quay.io/jetstack/trust-pkg-debian-trixie \
  --set defaultPackageImage.tag=20250419.2 \
  --wait

# Apply the corporate proxy CA source ConfigMap, then the Bundle that
# assembles + distributes the trust bundle
kubectl apply -f platform/trust-manager/manifests/corporate-proxy-ca.yaml
kubectl apply -f platform/trust-manager/manifests/bundle.yaml
```

The GitOps Application is
`platform/applications/trust-manager.yaml`. Its chart version and
`platform/trust-manager/values.yaml` must stay aligned with this bootstrap
command so ArgoCD adopts the existing Helm release rather than changing it
unexpectedly.

Once this and `platform/cert-manager` are both bootstrapped, continue with
the ArgoCD install steps in the top-level README.

## Verifying

```bash
kubectl get bundle aips-trust-bundle
# Confirm the ConfigMap landed in a sample of namespaces
kubectl get configmap aips-trust-bundle -n cert-manager -o jsonpath='{.data.ca-bundle\.crt}' | head -5
kubectl get configmap aips-trust-bundle -n kube-system -o name
```

## Updating the public CA package

`useDefaultCAs: true` loads public roots from the default CA package image;
it does not update the package independently of the Helm release. Review the
available tags in the
[trust-pkg-debian-trixie registry](https://quay.io/repository/jetstack/trust-pkg-debian-trixie?tab=tags)
regularly and update only `defaultPackageImage.tag` in
`values.yaml`. Keep the trust-manager chart version fixed while doing this.

For a manually bootstrapped release, export its values, update the package
repository and tag, and apply the currently installed controller version:

```bash
helm get values -n cert-manager trust-manager -o yaml > /tmp/trust-manager-values.yaml
# Edit /tmp/trust-manager-values.yaml, or copy the committed values.yaml into it.
TRUST_MANAGER_VERSION=$(helm list --filter '^trust-manager$' -n cert-manager -o json | jq -r '.[0].app_version')
helm upgrade trust-manager oci://quay.io/jetstack/charts/trust-manager \
  --install \
  --namespace cert-manager \
  --version "$TRUST_MANAGER_VERSION" \
  -f /tmp/trust-manager-values.yaml \
  --wait
```

With ArgoCD managing the release, update `values.yaml` in Git and let the
`trust-manager` Application sync. Verify the package and Bundle after either
path:

```bash
kubectl get bundle aips-trust-bundle -o yaml | grep -A5 -i default
kubectl get configmap aips-trust-bundle -n cert-manager -o jsonpath='{.data.ca-bundle\.crt}' | grep -c 'BEGIN CERTIFICATE'
```
