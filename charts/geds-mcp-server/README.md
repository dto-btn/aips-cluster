# GEDS MCP Server Helm Chart

This chart deploys the GEDS MCP Server for internal Kubernetes clients. It exposes the FastMCP Streamable HTTP endpoint at `/mcp` through a `ClusterIP` Service.

## Prerequisites

Create the required Secrets in the same namespace where the chart will be installed. Do not commit either Secret or the files containing their values to Git.

### API configuration Secret

Create a local, untracked env file containing the variables expected by the application:

```text
GEDS_API_ENDPOINT=https://api.geds-sage.gc.ca/gapi/v2
GEDS_API_TOKEN=replace-with-the-real-token
```

Create the Kubernetes Secret from that file:

```bash
kubectl create secret generic geds-mcp-server-config \
  --namespace <namespace> \
  --from-env-file=geds-mcp-server.env
```

The Secret must contain:

- `GEDS_API_ENDPOINT`
- `GEDS_API_TOKEN`

### CA bundle Secret

The application requires `/app/system_certs.pem` during startup. Create a Secret containing the CA bundle:

```bash
kubectl create secret generic geds-mcp-server-ca \
  --namespace <namespace> \
  --from-file=system_certs.pem=./system_certs.pem
```

The source `system_certs.pem` file must contain the certificates required to verify the GEDS API. The chart mounts it read-only at `/app/system_certs.pem`.

## Install

From the root of the `aips-cluster` repository:

```bash
helm upgrade --install geds-mcp-server ./charts/geds-mcp-server \
  --namespace <namespace> \
  --create-namespace \
  --set envFromSecret=geds-mcp-server-config \
  --set caBundle.existingSecret=geds-mcp-server-ca
```

The Secret names can also be set in a separate, uncommitted values file:

```yaml
envFromSecret: geds-mcp-server-config
caBundle:
  existingSecret: geds-mcp-server-ca
```

Do not put API keys or certificate contents in that values file.

## Internal connection

The default Service is a `ClusterIP` on port `8000`. Clients in the same namespace can use:

```text
http://geds-mcp-server:8000/mcp
```

Clients in another namespace can use:

```text
http://geds-mcp-server.<namespace>.svc.cluster.local:8000/mcp
```

The chart does not enable Ingress or Gateway API routes. It is not reachable from outside the cluster until external routing is configured.

## Validation

Render and inspect the manifests before installation:

```bash
helm lint ./charts/geds-mcp-server
helm template geds-mcp-server ./charts/geds-mcp-server \
  --set envFromSecret=geds-mcp-server-config \
  --set caBundle.existingSecret=geds-mcp-server-ca
```

After installation, verify the pod and Service:

```bash
kubectl get pods,svc --namespace <namespace>
kubectl describe pod --namespace <namespace> -l app.kubernetes.io/name=geds-mcp-server
```
