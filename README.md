# ArgoCD Playground

## Install with Helm

```bash
helm repo add argo https://argoproj.github.io/argo-helm
k create ns argocd
helm install my-argocd argo/argo-cd --version 7.7.17 -n argocd --debug
```

### Login to the ArgoCD UI
```bash

# Get the admin password generated
k -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
k -n argocd get secret argocd-initial-admin-secret -o json | jq .data.password -r | base64 -d

# Expose service locally
k -n argocd port-forward services/my-argocd-server 8080:80
```

### Reinstall
If required to reinstall

```bash

# Remove the Chart
helm -n argocd delete my-argocd

# Delete the CRDs created in the default namespace
k delete crd applications.argoproj.io
k delete crd applicationsets.argoproj.io
k delete crd appprojects.argoproj.io
```

### Other commands

```bash
# Check all resources created by argocd
k -n argocd get all

# Check the admin password encrypted
k -n argocd get secret argocd-secret -o json | jq '.data["admin.password"]' -r
k -n argocd get secret argocd-secret -o jsonpath="{.data.admin\.password}"

## Install ArgoCD CLI

```bash
brew intall argocd
```
