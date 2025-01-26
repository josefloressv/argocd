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
```

## ArgoCD CLI

### Install

```bash
# MacOS
brew intall argocd

# Check version
argocd version

```

### list applications

```bash
# Login using the exiting port-forwarding
argocd login 127.0.0.1:8080

# list the applications
argocd app list

# list clusters
argocd cluster list
```

## Manage applications

## Install Webapp color Helm app

```bash
# Create namespace
k create ns webapp

# Create an application project
argocd app create colorapp \
--repo https://github.com/josefloressv/helm \
--path webapp-color/ \
--dest-namespace webapp \
--dest-server https://kubernetes.default.svc

# List the applications
argocd app list

# Get info of the application
argocd app get colorapp

# Sync application
argocd app sync colorapp

```

## Install guest-book Helm app

* Repository: https://github.com/argoproj/argocd-example-apps
* Helm application path: helm-guestbook/

```bash
# Create namespace
k create ns guestbook

# Create an application project
argocd app create guestbook \
--repo https://github.com/argoproj/argocd-example-apps \
--path helm-guestbook/ \
--dest-namespace guestbook \
--dest-server https://kubernetes.default.svc \
--helm-set service.type=NodePort

# KK example
argocd app create solar-system-app-2 \
--repo https://3000-port-sekynz6nbi3fo364.labs.kodekloud.com/bob/gitops-argocd.git \
--path ./solar-system/ \
--dest-namespace solar-system \
--dest-server https://kubernetes.default.svc

```