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
│   ├── chart-agentgateway/              # Agent Gateway Controller (Helm Chart)
│   └── agentgateway-configs-application.yml # Argo CD App managing the configs/ folder
└── configs/                             # Your custom resources (auto-synced)
    ├── proxies/                         # Gateway Proxy definitions
    ├── mcp/                             # MCP Server workloads and routing
    ├── llm-providers/                   # LLM Provider configs (OpenAI, etc.)
    ├── agent-connectivity/              # Agent-to-Agent (A2A) and connectivity resources
    └── modules/                         # AgentgatewayParameters and module configs
```

## 🚀 Getting Started

### 1. Initialize your Repository
1.  Create a new Git repository (e.g., GitHub, GitLab).
2.  Push this folder's content to your repository.
3.  Update the `repoURL` placeholders with your actual Git repository URL (currently set to `https://github.com/OmerItach/agentgateway-infra.git`).

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
To set up a new proxy, create a Kubernetes `Gateway` resource in `configs/proxies/`.

### 🤝 Agent-to-Agent (A2A) Connectivity
To connect agents natively:
1.  Deploy your agent service with `appProtocol: kgateway.dev/a2a` on the port.
2.  Create an `HTTPRoute` in `configs/agent-connectivity/` pointing to your agent.
3.  Refer to the [A2A Connectivity guide](https://agentgateway.dev/docs/kubernetes/latest/agent/).

### 🤖 Adding LLM Providers
To integrate LLM providers like OpenAI:
1.  Create a `Secret` for the API key in `configs/llm-providers/`.
2.  Define an `AgentgatewayBackend` with the `ai` spec.
3.  Create an `HTTPRoute` pointing to the backend.
4.  Agent Gateway automatically handles URL rewriting (e.g., to `/v1/chat/completions`).
5.  Refer to the [LLM Consumption guide](https://agentgateway.dev/docs/kubernetes/latest/llm/).

### 🤖 Adding MCPs (Model Context Protocol)
MCP servers can be connected via `AgentgatewayBackend` and `HTTPRoute`. Place these in `configs/mcp/`.

### 🍱 Adding Modules (AgentgatewayParameters)
Agent Gateway allows fine-tuning via `AgentgatewayParameters`. Use the `configs/modules/` directory for these. These parameters allow you to configure LLM connectivity, global security policies, and more.

### 🔄 How to add a new "App" to the tree
If you want to add a new component (e.g., a monitoring stack):
1.  Create a new folder in `app-of-apps/`.
2.  Add an Argo CD `Application` manifest inside it.
3.  The `root-application.yaml` will automatically pick it up.

## 📋 Best Practices
- **Values Separation**: Keep sensitive data in external secrets (using external-secrets operator) and reference them in your manifests.
- **GitOps Flow**: Never use `kubectl apply` for your configs after the initial setup. Commit changes to Git and let Argo CD sync them.
- **Pruning**: Automated pruning is enabled in the root application to ensure your cluster state exactly matches your Git repository.
