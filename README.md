# Agent Gateway Infrastructure

This repository manages the deployment and configuration of [Agent Gateway](https://agentgateway.dev/) using the **App of Apps** pattern with Argo CD. It provides a production-grade infrastructure setup based on the best practices of the `homeProd-K3s` ecosystem.

## 📂 Project Structure

```
agentgateway-infra/
├── k8s/
│   ├── base/                            # Core manifests (Common to all envs)
│   │   ├── kustomization.yaml           # Base kustomization
│   │   ├── proxies/
│   │   ├── mcp/
│   │   ├── llm-providers/
│   │   ├── agent-connectivity/
│   │   └── modules/
│   └── overlays/                        # Environment-specific overrides
│       ├── dev/                         # Development Overlay
│       ├── stage/                       # Staging Overlay
│       └── prod/                        # Production Overlay
├── app-of-apps/                         # Argo CD Application Definitions
│   ├── root-application.yaml             # The "Master" application
│   └── ...                              # Component applications
├── repo.yml                             # Repo registration secret
└── README.md
```

## 🌳 Branching Strategy
Following the organization's standards:
- **`dev` branch**: Active development and the primary source for Argo CD.
- **`stage` branch**: For staging environment validation.
- **`main` branch**: Production-ready code.

> [!IMPORTANT]
> Per organization policy, Argo CD applications are configured to point to the `dev` branch and use path-based overlays to distinguish environments.

## 🚀 Getting Started

### 1. Initialize your Repository
1.  Push this content to your `dev` branch.
2.  Ensure `stage` and `main` branches are also initialized.

### 2. Register with Argo CD
```bash
kubectl apply -f repo.yml -n argocd
```

### 3. Deploy the Root Application
```bash
kubectl apply -f app-of-apps/root-application.yaml -n argocd
```

## 🛠 Usage & Extensions

### 🏘 Multi-Environment Config (Kustomize)
- **Base**: Add your core resources to `k8s/base/`.
- **Overlays**: Use `k8s/overlays/<env>/kustomization.yaml` to add patches or name prefixes for specific environments.

### 🤖 Adding Components
Follow the same patterns as before, but place your manifests in `k8s/base/` and add them to `k8s/base/kustomization.yaml`.

## 📋 Best Practices
- **Values Separation**: Keep sensitive data in external secrets (using external-secrets operator) and reference them in your manifests.
- **GitOps Flow**: Never use `kubectl apply` for your configs after the initial setup. Commit changes to Git and let Argo CD sync them.
- **Pruning**: Automated pruning is enabled in the root application to ensure your cluster state exactly matches your Git repository.
