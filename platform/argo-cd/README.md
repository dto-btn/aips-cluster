# ArgoCD

This is the ArgoCD installation for the platform that will manage the apps of apps pattern.

## Developper

Getting the credential and tunneling the dashboard:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
kubectl port-forward services/argocd-server -n argocd 8080:443
```