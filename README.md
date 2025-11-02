# Homelab GitOps with Argo CD (Showcase)

Production-style GitOps repo you can fork and point Argo CD at to demonstrate:
- App-of-apps bootstrap
- ApplicationSet (directory generator)
- AppProject guardrails
- Kustomize app with sync-waves & health checks
- Optional Helm-based infra (Ingress-NGINX)

> Target cluster: your homelab Kubernetes on ESXi (3x VMs). Argo CD is already installed.

## Repo layout

```
root/                 # App-of-apps (Argo CD Application) pulling everything below
projects/             # Argo CD AppProjects (RBAC & guardrails)
applicationsets/      # ApplicationSets for apps/ and infra/
apps/                 # Each folder = one app (Kustomize or Helm)
  podinfo/            # Sample app (Kustomize) with HPA and Service
infra/                # Optional cluster add-ons managed by Argo
  ingress-nginx/      # Helm-based NGINX Ingress controller
clusters/
  homelab/            # Cluster-specific config (placeholder)
bootstrap/            # One-time bootstrap manifest you apply to Argo CD
```

## Quick start (bootstrap Argo from this repo)

> Replace `<YOUR_GIT_URL>` with your fork (HTTPS or SSH). If private, add repo creds in Argo CD first.

```bash
# 1) Point this bootstrap application at your fork
# Edit bootstrap/argocd-root-app.yaml: spec.source.repoURL: <YOUR_GIT_URL>

# 2) Apply the root "app-of-apps" to your existing Argo CD install:
kubectl -n argocd apply -f bootstrap/argocd-root-app.yaml

# 3) Watch apps appear
kubectl -n argocd get applications
```

### Optional: expose Argo CD server via NodePort
If you don't already have Ingress for Argo, you can port-forward for a quick demo:
```bash
kubectl -n argocd port-forward svc/argocd-server 8443:443
# then browse https://localhost:8443
```

## What this demonstrates

- **AppProject guardrails**: only allow deployment to `homelab` namespaces, only from this repo, and restrict cluster-scoped resources via resource allowlist.
- **ApplicationSet**: auto-creates an Argo CD Application per subfolder under `apps/` and `infra/` (git directory generator).
- **Kustomize app**: the sample `apps/podinfo` includes a Deployment, Service, HPA, and sync-wave annotations so Services come up before Deployments.
- **Helm app**: `infra/ingress-nginx` shows Argo managing a third-party chart.
- **Multi-env ready**: Add more clusters or envs later by adding folders and a new ApplicationSet/cluster generator.

## Add your own app

1. Create `apps/myapp/` with either Kustomize (recommended) or a Helm `application.yaml` referencing a remote chart.
2. Commit & push.
3. ApplicationSet will auto-create an `Application` named `myapp` and sync it.

## Clean up

Delete the root application and Argo will cascade-delete managed apps:
```bash
kubectl -n argocd delete application argocd-root
```
