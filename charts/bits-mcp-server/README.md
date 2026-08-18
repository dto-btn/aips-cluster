# Business Requests MCP Server Helm Chart

This chart deploys the Business Requests MCP server for internal Kubernetes clients. It exposes the FastMCP Streamable HTTP endpoint at `/mcp` through a `ClusterIP` Service and connects to the external SQL Server database configured for BITS.

## Prerequisites

The Kubernetes cluster must be able to resolve and reach the database server. The application uses ODBC Driver 18 and encrypted SQL Server connections with certificate verification.

### Azure SQL firewall

Before testing a database-backed MCP tool, add the Kubernetes workload's public egress IPv4 address to the firewall rules on the Azure SQL logical server. Azure SQL does not see the pod or Minikube private IP; it sees the public IP from which the connection exits the network.

Use the exact address reported in a failed SQL connection, or check the current egress address from the pod:

```bash
kubectl exec --namespace bits-dev deploy/bits-mcp-server -- \
  python -c "import urllib.request; print(urllib.request.urlopen('https://api.ipify.org').read().decode())"
```

For a local test, add that single IPv4 address as both the firewall rule's start and end address. Do not whitelist pod IPs such as `10.244.x.x`, Minikube addresses, or a broad internet-wide range. Public IPs can change when the network, location, VPN, router, or ISP changes, so recheck the egress address when connections begin failing again.

Allow several minutes for an Azure SQL firewall change to take effect.

## Database Secret

Create a local, untracked env file containing the server configuration:

```text
BITS_DB_SERVER=your-db-server.domain
BITS_DB_USERNAME=your-username
BITS_DB_PWD=your-password
BITS_DB_DATABASE=your-database-name
```

The server Secret must contain these four keys. `ODBC_DRIVER` may also be added if the default `{ODBC Driver 18 for SQL Server}` is not appropriate.

Because the application uses FastMCP's settings, include these server settings in the Secret as well:

```text
FASTMCP_HOST=0.0.0.0
FASTMCP_PORT=8000
```

Do not include the client-only Azure OpenAI or experimental OAuth variables. They are not required by `server.py`.

Create the Secret in the deployment namespace:

```bash
kubectl create namespace bits-dev
kubectl create secret generic bits-db-config \
  --namespace bits-dev \
  --from-env-file=bits-server.env
```

Do not commit `bits-server.env` or the database credentials to Git.

## Install

From the root of the `aips-cluster` repository:

```bash
helm upgrade --install bits-mcp-server \
  ./charts/bits-mcp-server \
  --namespace bits-dev \
  --set envFromSecret=bits-db-config
```

The chart uses a `ClusterIP` Service on port `8000`. Ingress and Gateway API routes are disabled by default.

## Internal connection

Clients in the same namespace can use:

```text
http://bits-mcp-server:8000/mcp
```

Clients in another namespace can use:

```text
http://bits-mcp-server.bits-dev.svc.cluster.local:8000/mcp
```

For local MCP Inspector testing:

```bash
kubectl port-forward --namespace bits-dev \
  service/bits-mcp-server 8000:8000
```

Connect the Inspector to:

```text
http://127.0.0.1:8000/mcp
```

## Validation

```bash
helm lint ./charts/bits-mcp-server
helm template bits-mcp-server ./charts/bits-mcp-server \
  --set envFromSecret=bits-db-config
helm test bits-mcp-server --namespace bits-dev
```

Check pod startup and database connection errors with:

```bash
kubectl get pods --namespace bits-dev
kubectl logs --namespace bits-dev \
  -l app.kubernetes.io/instance=bits-mcp-server
kubectl get events --namespace bits-dev --sort-by=.lastTimestamp
```

The initial resource settings are conservative starting points:

- Requests: `100m` CPU and `256Mi` memory
- Limits: `500m` CPU and `512Mi` memory

Refine them after observing actual query workload and pod metrics.