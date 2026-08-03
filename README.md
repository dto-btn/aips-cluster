# AI Platform Solutions CLuster(s)

This repo contains all the platform and instance Manifests to maintain via GitOps the AIPS Cluster.

## Initial Cluster Setup

This will explain how to initiall bootstrap and work with this repository

### Step 1

The first step assumes that [the repository containing the `terraform` code for AKS has been applied]().

### Step 2

This instruct on how to run the first `kubectl` commands needed to finish the cluster initialisation.

* Login webtop inside the `G3Pc` envionrment VM,
* bootstrap the context for `kubectl` (install that binary if not already done) via `az aks get-credentials` (this will store the config inside `~/.kube/config`)
* Install argo CD:
  ```bash
  kubectl create namespace aips-platform
  kubectl apply -n aips-platform --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  ```

### Step 3

Manual bootstrap of the `instances/application/app-deployer.yaml` and `platform/applications/app-deployer.yaml` **app of apps** pattern. via: 

```bash
cd ~/github/aips-cluster/ 
kubectl apply -f platform/applications/app-deployer.yaml
kubectl apply -f instances/applications/app-deployer.yaml
```

## Documentation and info

* This repo layout is based on the current CANChat [repo layout located here](https://github.com/ssc-dsai-iac/canchat-thinkon-prod)