# AI Platform Solutions CLuster(s)

This repo contains all the platform and instance Manifests to maintain via GitOps the AIPS Cluster.

## Initial Cluster Setup

This will explain how to initiall bootstrap and work with this repository

### Step 1

The first step assumes that [the repository containing the `terraform` code for AKS has been applied]().

### Step 2

This instruct on how to run the first `kubectl` commands needed to finish the cluster initialisation.

* Login webtop inside the `G3Pc` envionrment VM,
* bootstrap the context for `kubectl` (install that binary if not already done) via `az aks get-credentials --admin --resource-group G3Pc-SSC_EPSB_AKS-rg --name AIPS_Cluster --file ~/.kube/config` (this will store the config inside `~/.kube/config`)
* Install argo CD:s
  ```bash
  kubectl create namespace aips-platform
  kubectl create namespace argocd
  kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  ```

### Step 3

Manual bootstrap of the `instances/application/app-deployer.yaml` and `platform/applications/app-deployer.yaml` **app of apps** pattern. via: 

```bash
cd ~/github/aips-cluster/
# TODO
# helm add argo...
# helm install argo chart with value file...
# then
kubectl apply -f platform/argo-cd/projects/aips-platform.yaml
kubectl apply -f platform/applications/app-deployer.yaml


#current state.  Applied certs to values.yaml.  But returning a 403.  not sure what's going on =/
#not applied yet.
kubectl apply -f instances/prod/applications/app-deployer.yaml
# eventually ...
# kubectl apply -f instances/dev/applications/app-deployer.yaml
```

## Repository Structure

> ⚠️ **Work in progress**: this repo is still being built out. The tree below shows the
> current state plus the intended shape (marked `WIP`) so the next dev knows where things
> are headed, not just where they are today.

```bash
aips-cluster/
├── charts/                        # Local Helm charts owned/maintained by this repo (not 3rd-party)
│   └── ssca-mcp-server/           # e.g. a chart for a service without an upstream chart
│
├── platform/                      # Cluster-wide, shared infra — deployed once, used by every instance
│   ├── applications/               # ArgoCD "app of apps" root for the platform namespace (aips-platform)
│   │   ├── app-deployer.yaml       # Bootstraps/syncs all platform Applications from source
│   │   └── argo-cd.yaml            # ArgoCD deploys/manages itself via this Application
│   ├── argo-cd/
│   │   ├── manifests/              # Raw k8s manifests applied alongside the argo-cd Helm chart
│   │   └── projects/               # AppProject definitions (RBAC/scoping per instance, e.g. ekh-prod, ssca-prod)
│   └── cert-manager/               # WIP: another platform-level tool, same manifests/ pattern
│       └── manifests/
│
└── instances/                     # Actual product/app deployments, split per environment
    ├── dev/                        # WIP: mirrors prod/ structure once dev is bootstrapped
    │   └── README.md
    └── prod/                       # Production environment (namespace aips-prod)
        ├── README.md
        └── applications/           # ArgoCD "app of apps" root for this environment
            ├── app-deployer.yaml    # Bootstraps/syncs all Applications listed in this folder
            ├── ekh.yaml             # One ArgoCD Application per product (Enterprise Knowledge Hub)
            └── ssca-frontend.yaml   # One ArgoCD Application per product (SSCA frontend)
```

**Key ideas for whoever picks this up:**

- **`platform/`** = shared cluster tooling (ArgoCD itself, cert-manager, ingress, etc.) —
  things every instance depends on but that aren't a "product". Deployed once into
  `aips-platform`.
- **`instances/`** = one folder per environment (`dev`, `prod`, ...), each with its own
  `applications/` app-of-apps that lists the actual products running there (e.g. `ekh`,
  `ssca-frontend`). This is where product teams' Applications get added.
- **`app-deployer.yaml`** is the recurring pattern: an ArgoCD Application pointed at its own
  parent folder, so ArgoCD keeps that folder's Applications in sync with git automatically.
- **`charts/`** holds Helm charts we author ourselves, referenced by Applications above via
  `repoURL`/`path` when no suitable upstream chart exists.
- Each new product/environment should follow the existing `ekh.yaml` / `ssca-frontend.yaml`
  pattern: a small ArgoCD `Application` manifest dropped into the relevant
  `instances/<env>/applications/` folder, plus an `AppProject` under
  `platform/argo-cd/projects/` to scope its RBAC and allowed source repos.

## Documentation and info

* This repo layout is based on the current CANChat [repo layout located here](https://github.com/ssc-dsai-iac/canchat-thinkon-prod)