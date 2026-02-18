# Agent Gateway Infrastructure

This repository manages the deployment and configuration of [Agent Gateway](https://agentgateway.dev/) using the **App of Apps** pattern with Argo CD. It provides a production-grade infrastructure setup based on the best practices of the `homeProd-K3s` ecosystem.

## 📂 Project Structure

```
agentgateway-infra/
├── repo.yml                             # Template for Argo CD Repo registration
├── README.md                            # This file
├── app-of-apps/                         # Infrastructure Application Definitions
│   ├── root-application.yaml             # The "Master" application (Root of the tree)
│   ├── gateway-api/                     # Kubernetes Gateway API CRDs (v1.4.0)
│   ├── chart-agentgateway-crds/         # Agent Gateway specific CRDs
│   └── chart-agentgateway/              # Agent Gateway Controller (Helm Chart)
│       ├── agentgateway-application.yml # Argo CD Application manifest
│       └── values.yaml                  # Helm values (Experimental features enabled)
└── configs/                             # (Recommended) Place for your custom resources
```

## 🚀 Getting Started

### 1. Initialize your Repository
1.  Create a new Git repository (e.g., GitHub, GitLab).
2.  Push this folder's content to your repository.
3.  Update the `repoURL` placeholders in the following files:
    - `repo.yml`
    - `app-of-apps/root-application.yaml`
    - `app-of-apps/chart-agentgateway/agentgateway-application.yml`

### 2. Register with Argo CD
Apply the repository secret to your Argo CD namespace:
```bash
kubectl apply -f repo.yml -n argocd
```

### 3. Deploy the Root Application
Apply the root application to start the sync process:
```bash
kubectl apply -f app-of-apps/root-application.yaml -n argocd
```

## 🛠 Usage & Extensions

This project is designed to be easily extended with new configurations.

### 🌐 Adding Proxies (Gateways)
To set up a new proxy, create a Kubernetes `Gateway` resource. It is recommended to place these in a `configs/proxies/` folder.

**Example `Gateway`:**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: agent-proxy
  namespace: agentgateway-system
spec:
  gatewayClassName: agentgateway
  listeners:
  - protocol: HTTP
    port: 80
    name: http
```

### 🤖 Adding MCPs (Model Context Protocol)
MCP servers can be connected via `HTTPRoute` or `GRPCRoute` targeting your MCP backend services.

1.  Create an `HTTPRoute` that matches your MCP server path.
2.  Deploy the manifest under `configs/mcp/`.
3.  Refer to the [MCP Documentation](https://agentgateway.dev/docs/kubernetes/latest/mcp/) for specific static vs dynamic configurations.

### 🍱 Adding Modules (AgentgatewayParameters)
Agent Gateway allows fine-tuning via `AgentgatewayParameters`. This is used to configure specific modules or global controller settings.

**Example:**
```yaml
apiVersion: agentgateway.dev/v1alpha1
kind: AgentgatewayParameters
metadata:
  name: global-config
spec:
  # Module specific configurations here
```

### 🔄 How to add a new "App" to the tree
If you want to add a new component (e.g., a monitoring stack specific to Agent Gateway):
1.  Create a new folder in `app-of-apps/`.
2.  Add an Argo CD `Application` manifest inside it.
3.  The `root-application.yaml` is configured with `recurse: true`, so it will automatically pick up the new application and deploy it.

## 📋 Best Practices
- **Values Separation**: Keep sensitive data in external secrets (using external-secrets operator) and reference them in your manifests.
- **GitOps Flow**: Never use `kubectl apply` for your configs after the initial setup. Commit changes to Git and let Argo CD sync them.
- **Pruning**: Automated pruning is enabled in the root application to ensure your cluster state exactly matches your Git repository.
