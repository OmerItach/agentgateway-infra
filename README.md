# 🤖 Agent Gateway Infrastructure (GitOps)

This repository manages the deployment of **Agent Gateway** using **Argo CD** and the **App-of-Apps** pattern. It is structured to support multi-environment deployments using Kustomize overlays.

## 🏗 Repository Structure

```text
agentgateway-infra/
├── k8s/
│   ├── base/                            # Core manifests (Common to all envs)
│   │   ├── kustomization.yaml           # Base kustomization (indexes all components)
│   │   ├── proxies/                     # Gateway Proxy (The entry point)
│   │   ├── mcp/                         # MCP Server workloads and routing
│   │   ├── llm-providers/               # LLM Provider configs (OpenAI, Gemini, etc.)
│   │   ├── agent-connectivity/          # Agent-to-Agent (A2A) resources
│   │   └── modules/                     # AgentgatewayParameters & global configs
│   └── overlays/                        # Environment-specific overrides
│       ├── dev/                         # Development Environment (Namespace: agentgateway-system)
│       ├── stage/                       # Staging Environment (Namespace: agentgateway-stage)
│       └── prod/                        # Production Environment (Namespace: agentgateway-prod)
├── app-of-apps/                         # Argo CD Application Definitions
│   ├── root-application.yaml            # The entry point for the entire stack
│   └── ...                              # Component applications (Helm, Configs, etc.)
├── repo.yml                             # Argo CD Repository Registration
└── README.md                            # This file
```

## 🌳 Branching & Workflow

Following organization standards, this repository uses the following branch strategy:
- **`dev` branch**: Primary source of truth for all environments.
- **`stage` branch**: Mirrored from `dev` for higher environment validation.
- **`main` branch**: Production-ready manifests.

> [!TIP]
> **GitOps Standard**: Argo CD Applications should point to the **`dev` branch** and use the **path** field to distinguish between environment overlays (`k8s/overlays/dev`, etc.).

---

## 🚀 Deployment Guide

### 1. Register the Repository
Argo CD needs access to this private (or public) repository. Apply the registration secret:
```bash
kubectl apply -f repo.yml -n argocd
```

### 2. Deploy a Specific Environment
To deploy the **Dev** environment, ensure the `root-application.yaml` points to the `dev` overlay. To deploy to **Stage** or **Prod**, you can create separate Root Applications in Argo CD with different target paths:

| Environment | Repo Branch | Path | Target Cluster |
| :--- | :--- | :--- | :--- |
| **Dev** | `dev` | `k8s/overlays/dev` | `https://kubernetes.default.svc` |
| **Stage** | `dev` | `k8s/overlays/stage` | `https://stage-cluster-ip` |
| **Prod** | `dev` | `k8s/overlays/prod` | `https://prod-cluster-ip` |

### 3. Deploy to Different Clusters
To deploy an overlay to a remote cluster via Argo CD:
1.  Add the remote cluster to Argo CD: `argocd cluster add <context-name>`.
2.  In the `Application` manifest (under `app-of-apps/`), update the `destination.server` field to the remote cluster's URL.

---

## 🛠 How to Add New Components

### 🤖 Adding a new LLM Provider
1.  Create a spec file in `k8s/base/llm-providers/<name>.yaml`.
2.  Define a `Secret`, `AgentgatewayBackend`, and `HTTPRoute`.
3.  **Crucial**: Add a unique path match to the `HTTPRoute` to avoid 404 conflicts:
    ```yaml
    rules:
      - matches:
        - path: { type: PathPrefix, value: /<unique-name> }
        backendRefs: ...
    ```
4.  Add the new file to `k8s/base/kustomization.yaml`.

### 🤝 Connecting a new MCP Server
1.  Add the Deployment/Service manifests to `k8s/base/mcp/`.
2.  Create an `AgentgatewayBackend` and `HTTPRoute` in `k8s/base/mcp/mcp-routing.yaml`.
3.  Use **relative hostnames** (e.g., `mcp-service-name`) so Kustomize can handle namespace shifts automatically.

### 🍱 Overriding Configs per Environment
If you need to use a different API key or replica count for **Prod**:
1.  Go to `k8s/overlays/prod/kustomization.yaml`.
2.  Use `patchesJson6902` or `patchesStrategicMerge` to modify the base manifests without changing the original files.

---

## 🔍 Troubleshooting Fixes
- **404 Errors**: Usually caused by multiple `HTTPRoute` resources lacking unique `matches.path` prefixes. Always ensure every route has a distinct path (e.g., `/openai`, `/mcp`).
- **Service Discovery**: Use the name of the Kubernetes Service as the `host` in `AgentgatewayBackend`. Avoid FQDNs like `.svc.cluster.local` in `base/` to ensure portability across namespaces.
