# ARC runner scale set

This runner scale set uses a custom Docker-in-Docker (`dind`) template instead
of the chart's built-in `containerMode.type: dind` template so that the Docker
daemon can be started with cluster-specific MTU settings.

## Docker MTU

The actions-runner-controller troubleshooting guide documents a failure mode
where outgoing network operations can hang indefinitely when Docker assumes the
standard MTU of `1500`, but the Kubernetes pod network uses a smaller MTU:

<https://github.com/actions/actions-runner-controller/blob/master/TROUBLESHOOTING.md#outgoing-network-action-hangs-indefinitely>

That guide shows `dockerMTU` for the older `RunnerDeployment` API. This
configuration uses the newer `gha-runner-scale-set` Helm chart and a manually
customized `dind` container, so the equivalent setting is passed directly to
`dockerd`:

```yaml
args:
  - dockerd
  - --host=unix:///var/run/docker.sock
  - --group=$(DOCKER_GROUP_GID)
  - --mtu=1300
  - --network-control-plane-mtu=1300
```

The runner image also includes an ARC Docker shim that can copy the default
bridge MTU onto Docker networks created by the GitHub runner for service
containers and container jobs. Keep this enabled on the runner container:

```yaml
env:
  - name: ARC_DOCKER_MTU_PROPAGATION
    value: "true"
```

The value should match the MTU of the outgoing interface inside a runner pod.
To confirm the value in the cluster, run:

```sh
kubectl -n arc-runner-dto-btn exec -it <runner-pod> -c runner -- ip link
```

If the pod interface MTU changes, update both Docker daemon flags in
`values.yaml` to the new MTU.

## Why DinD is used

The `dind` sidecar runs the Docker daemon for the GitHub Actions runner. The
runner container connects to it through:

```yaml
DOCKER_HOST: unix:///var/run/docker.sock
```

This allows workflows to run Docker commands and supports workflow features that
need containers, such as Docker container actions, service containers, and jobs
configured with `container:`.
