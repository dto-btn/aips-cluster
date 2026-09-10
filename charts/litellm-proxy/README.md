# LiteLLM Proxy Helm Chart

This Helm chart deploys the `litellm-proxy` service in Kubernetes, configured with custom callback hooks for structured logging and optional in-cluster or external PostgreSQL persistence.

---

## Architecture Overview

- **LiteLLM Proxy Deployment**: Runs the custom LiteLLM container on port `4000`, exposed internally via a Kubernetes Service on port `80`.
- **ConfigMap (`config.prod.yaml`)**: Managed via Helm (`configMap.enabled: true`), allowing model lists, router settings, and custom callback configurations to be updated without rebuilding container images.
- **Environment Secrets (`existingSecret`)**: Reads sensitive runtime variables (such as API keys and master keys) from a pre-created Kubernetes Secret (`litellm-proxy-secrets`).
- **PostgreSQL Database (`postgres.enabled`)**:
  - **In-Cluster (Default)**: Deploys a single-node PostgreSQL `StatefulSet` with persistent storage (`PVC`).
  - **External / Cloud-Managed**: Can be disabled to connect directly to an external managed database (e.g., Azure Database for PostgreSQL).

---

## Database & Secret Configuration Setup

Before deploying the chart, create a Kubernetes Secret (`litellm-proxy-secrets`) in your target namespace from an `.env` file (`--from-env-file`).

The setup depends on whether you are using the **In-Cluster PostgreSQL** (recommended for local dev and staging) or an **External / Managed PostgreSQL** (recommended for production).

---

### Option A: In-Cluster PostgreSQL Setup (Local Dev & Testing)

In this mode, `postgres.enabled` is `true` (default). Helm provisions a local PostgreSQL `StatefulSet` inside the cluster, and both LiteLLM and Postgres read database credentials from your secret.

1. **Copy [.env.example](.env.example) to `.env.local`**:
   ```bash
   cp .env.example .env.local
   ```

2. **Populate `.env.local`**:
   ```env
   # Core LiteLLM Configuration
   LITELLM_MASTER_KEY=sk-1234...
   AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
   AZURE_OPENAI_API_KEY=your-azure-key
   AZURE_OPENAI_VERSION=2025-03-01-preview
   LITELLM_DEFAULT_MODEL=azure/gpt-4o

   # Database Configuration (In-Cluster Postgres)
   POSTGRES_DB=litellm
   POSTGRES_USER=litellm
   POSTGRES_PASSWORD=your_secure_postgres_password
   ```

3. **Create Secret in Kubernetes**:
   ```bash
   kubectl create namespace litellm
   kubectl create secret generic litellm-proxy-secrets --from-env-file=.env.local -n litellm
   ```

---

### Option B: External / Managed Database Setup (Recommended for Production)

In production environments, it is strongly recommended to use a managed, high-availability database (such as **Azure Database for PostgreSQL Flexible Server**) rather than an in-cluster `StatefulSet`.

1. **Copy [.env.prod.example](.env.prod.example) to `.env.prod`**:
   ```bash
   cp .env.prod.example .env.prod
   ```

2. **Populate `.env.prod`**:
   ```env
   # Core LiteLLM Configuration
   LITELLM_MASTER_KEY=sk-1234...
   AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
   AZURE_OPENAI_API_KEY=your-azure-key
   AZURE_OPENAI_VERSION=2025-03-01-preview
   LITELLM_DEFAULT_MODEL=azure/gpt-4o

   # Database Configuration for External/Managed Postgres
   DATABASE_URL=postgresql://litellm_user:<password>@<your-pg-server>.postgres.database.azure.com:5432/litellm?sslmode=require
   ```

3. **Create or Update Secret in Kubernetes**:
   ```bash
   kubectl create namespace litellm
   kubectl create secret generic litellm-proxy-secrets \
     --from-env-file=.env.prod \
     -n litellm \
     --dry-run=client -o yaml | kubectl apply -f -
   ```

4. **Disable In-Cluster PostgreSQL in Helm / Argo CD**:
   Set `postgres.enabled: false` in your release values:
   ```yaml
   postgres:
     enabled: false
   ```

   When `postgres.enabled` is `false`:
   - Helm skips generating the in-cluster Postgres `StatefulSet`, `Service`, `PVC`, and DB secrets.
   - LiteLLM Proxy automatically reads `DATABASE_URL` from `litellm-proxy-secrets`.

---

## Local Testing & Deployment Guide

Follow these steps to deploy and test the chart locally (e.g. using `minikube` or `kind`):

### 1. Build or Load the Container Image
If using Minikube, load your locally built image into the Minikube cluster node:

```bash
minikube image load ectacr.azurecr.io/litellm-proxy:1.0.1
```

### 2. Create Target Namespace
```bash
kubectl create namespace litellm
```

### 3. Create the Secret from Environment File
Prepare your `.env.local` file from `.env.example`, then create the secret in the `litellm` namespace:

```bash
cp .env.example .env.local
# Edit .env.local with your API keys and DB credentials
kubectl create secret generic litellm-proxy-secrets --from-env-file=.env.local -n litellm
```

### 4. Install the Helm Chart
Run the Helm install command targeting the `litellm` namespace:

```bash
helm install litellm-proxy ./charts/litellm-proxy -n litellm
```

### 5. Verify Deployment & Pod Status
Check that both the PostgreSQL StatefulSet and LiteLLM Proxy Deployment pods are running:

```bash
kubectl get pods -n litellm
kubectl get svc -n litellm
```

### 6. Test Connectivity Locally
Port-forward the LiteLLM Proxy service to your local machine:

```bash
kubectl port-forward svc/litellm-proxy 4000:80 -n litellm
```

In a separate terminal, verify the health endpoints and test a completion request using curl:

```bash
# Check health readiness
curl http://localhost:4000/health/readiness

# Test chat completions endpoint (using LITELLM_MASTER_KEY)
curl -X POST http://localhost:4000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-1234..." \
  -d '{
    "model": "gpt-4o",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### 7. Run Helm Test
Execute the built-in Helm connection test hook:

```bash
helm test litellm-proxy -n litellm
```

---

## Configuration Reference

Key values in `values.yaml`:

| Parameter | Description | Default |
| :--- | :--- | :--- |
| `service.port` | External Service port | `80` |
| `service.targetPort` | Container port for LiteLLM | `4000` |
| `existingSecret` | Name of secret containing API keys / master keys | `"litellm-proxy-secrets"` |
| `configMap.enabled` | Manage `config.prod.yaml` via ConfigMap | `true` |
| `postgres.enabled` | Enable in-cluster PostgreSQL StatefulSet | `true` |
| `postgres.auth.secretKeys.database` | Key for DB name in secret | `"POSTGRES_DB"` |
| `postgres.auth.secretKeys.user` | Key for DB user in secret | `"POSTGRES_USER"` |
| `postgres.auth.secretKeys.password` | Key for DB password in secret | `"POSTGRES_PASSWORD"` |
| `postgres.persistence.size` | Storage volume size for Postgres PVC | `"10Gi"` |
