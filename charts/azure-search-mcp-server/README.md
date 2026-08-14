# Azure Search MCP Server Helm Chart

This chart deploys the Azure AI Search MCP server for internal Kubernetes clients. The server exposes the Streamable HTTP MCP endpoint at `/mcp` and connects to one configured Azure AI Search index per Helm release.

Use this chart for both PMCOE and MySSC+ by installing it twice with different release names, namespaces, and existing Secrets.

## Required Secret

Each release needs an existing Kubernetes Secret in its target namespace. The Secret must contain:

```text
AZURE_SEARCH_SERVICE_ENDPOINT
AZURE_SEARCH_INDEX_NAME
AZURE_SEARCH_API_KEY
```

Optional index-specific settings can be included in the same Secret:

```text
AZURE_SEARCH_KEY_FIELD
AZURE_SEARCH_CONTENT_FIELD
AZURE_SEARCH_VECTOR_FIELD
AZURE_SEARCH_SEMANTIC_CONFIG
```

The application also supports optional non-secret settings such as `SEARCH_TOP_RESULTS`, `MCP_SERVER_NAME`, `CORS_ALLOW_ORIGINS`, and `CORS_ALLOW_CREDENTIALS` through the same Secret.

Do not commit the source env files or API keys to Git.

## Create the Secrets

Create each Secret from its local, untracked env file. Run these commands from the Azure Search MCP server repository, where the configuration files are located:

```bash
kubectl create namespace pmcoe-mcp
kubectl create secret generic azure-search-pmcoe-config \
  --namespace pmcoe-mcp \
  --from-env-file=configs/pmcoe.env
```

```bash
kubectl create namespace myssc-mcp
kubectl create secret generic azure-search-myssc-config \
  --namespace myssc-mcp \
  --from-env-file=configs/myssc.env
```

Use a secret-management system instead of `kubectl create secret` for shared or production environments.

## Install both releases

Run these commands from the root of the `aips-cluster` repository:

```bash
helm upgrade --install azure-search-pmcoe \
  ./charts/azure-search-mcp-server \
  --namespace pmcoe-mcp \
  --set envFromSecret=azure-search-pmcoe-config
```

```bash
helm upgrade --install azure-search-myssc \
  ./charts/azure-search-mcp-server \
  --namespace myssc-mcp \
  --set envFromSecret=azure-search-myssc-config
```

The release names make the Deployment and Service names unique even though both releases use the same chart.

## Internal MCP endpoints

The Services are `ClusterIP` Services. Clients in the same namespace can use:

```text
http://azure-search-pmcoe-azure-search-mcp-server:8000/mcp
http://azure-search-myssc-azure-search-mcp-server:8000/mcp
```

Clients in another namespace can use the corresponding Kubernetes DNS names:

```text
http://azure-search-pmcoe-azure-search-mcp-server.pmcoe-mcp.svc.cluster.local:8000/mcp
http://azure-search-myssc-azure-search-mcp-server.myssc-mcp.svc.cluster.local:8000/mcp
```

For local testing, port-forward either Service:

```bash
kubectl port-forward --namespace pmcoe-mcp \
  service/azure-search-pmcoe-azure-search-mcp-server 8000:8000
```

Then connect an MCP client to:

```text
http://127.0.0.1:8000/mcp
```

## Validation

```bash
helm lint ./charts/azure-search-mcp-server
helm template azure-search-pmcoe ./charts/azure-search-mcp-server \
  --set envFromSecret=azure-search-pmcoe-config
helm test azure-search-pmcoe --namespace pmcoe-mcp
```

Check startup failures with:

```bash
kubectl get pods --namespace pmcoe-mcp
kubectl logs --namespace pmcoe-mcp \
  -l app.kubernetes.io/instance=azure-search-pmcoe
```

The application performs Azure Search connection and index metadata discovery during startup. Invalid credentials, an unavailable endpoint, a missing index, or incompatible index fields can therefore prevent the pod from becoming ready.
