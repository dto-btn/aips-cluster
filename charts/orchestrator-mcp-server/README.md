# SSCA Orchestrator MCP Server Helm Chart

This chart deploys the SSCA orchestrator MCP server for internal Kubernetes clients. It classifies requests using the baked-in registry and deterministic keyword routing. LLM classification through LiteLLM is disabled for this first iteration.

## Install

From the root of the `aips-cluster` repository:

```bash
kubectl create namespace orchestrator-dev

helm upgrade --install orchestrator-mcp-server \
  ./charts/orchestrator-mcp-server \
  --namespace orchestrator-dev
```

No Secret is required for this iteration. The chart sets:

```text
ENABLE_LLM_CLASSIFIER=false
ORCHESTRATOR_HOST=0.0.0.0
ORCHESTRATOR_PORT=8000
```

The image starts Uvicorn on `0.0.0.0:8000`, and the Service is an internal `ClusterIP` on port `8000`.

## Endpoints

The Streamable HTTP MCP endpoint is:

```text
/mcp
```

The HTTP helper endpoint is:

```text
POST /orchestrator/suggest-route
```

For local MCP Inspector testing:

```bash
kubectl port-forward --namespace orchestrator-dev \
  service/orchestrator-mcp-server 8000:8000
```

Connect Inspector to:

```text
http://127.0.0.1:8000/mcp
```

Test the HTTP helper directly:

```bash
curl -s http://127.0.0.1:8000/orchestrator/suggest-route \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{"role": "user", "content": "Find an employee in the government directory"}],
    "max_recommendations": 3,
    "require_single_best": false
  }'
```

## Cross-namespace communication

Kubernetes Services can communicate across namespaces using the Service DNS name. For this release, clients in another namespace can use:

```text
http://orchestrator-mcp-server.orchestrator-dev.svc.cluster.local:8000/mcp
```

The short name `orchestrator-mcp-server` resolves automatically only for clients in the same namespace. NetworkPolicies or other cluster firewall rules may still restrict cross-namespace traffic.

## Registry

The current registry is copied into the image at `/app/mcp_registry.json` and loaded by the default setting:

```text
ORCHESTRATOR_REGISTRY_PATH=./mcp_registry.json
```

The registry contains downstream MCP server IDs, endpoints, categories, keywords, and routing rules. Keep it baked into the image for this iteration.

## Validation and troubleshooting

```bash
helm lint ./charts/orchestrator-mcp-server
helm template orchestrator-mcp-server ./charts/orchestrator-mcp-server
helm test orchestrator-mcp-server --namespace orchestrator-dev
kubectl get pods,svc --namespace orchestrator-dev
kubectl logs --namespace orchestrator-dev \
  -l app.kubernetes.io/instance=orchestrator-mcp-server
```

The initial chart uses TCP startup, readiness, and liveness probes because the application does not expose a dedicated health endpoint. Inspect events if a pod is not Ready:

```bash
kubectl get events --namespace orchestrator-dev --sort-by=.lastTimestamp
```

## Next iteration

When the LiteLLM proxy is available:

1. Enable LLM classification with `ENABLE_LLM_CLASSIFIER=true`.
2. Configure `ORCHESTRATOR_LITELLM_PROXY_URL` and `ORCHESTRATOR_LLM_MODEL`.
3. Store `ORCHESTRATOR_LITELLM_PROXY_API_KEY` and/or `ORCHESTRATOR_LITELLM_PROXY_BEARER_TOKEN` in a Kubernetes Secret.
4. Add chart support for `envFromSecret` and verify proxy authentication from inside the cluster.
5. Decide whether the proxy URL should be a cross-namespace Service DNS name, such as `litellm-proxy.<namespace>.svc.cluster.local:4000/v1`.

For registry updates without rebuilding the image, add a ConfigMap containing `mcp_registry.json`, mount it at `/app/mcp_registry.json`, and set `ORCHESTRATOR_REGISTRY_PATH` accordingly. This should be introduced only after deciding who owns and secures registry changes.

External Ingress/Gateway routing and client authentication should be added after internal MCP and LiteLLM integration tests pass.