# ArgoCD Playground

## Install with Helm

```bash
helm repo add argo https://argoproj.github.io/argo-helm
k create ns argocd
helm install my-argocd argo/argo-cd --version 7.7.17 -n argocd --debug
# configs.secret.argocdServerAdminPassword
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
# Get initial password
argocd admin initial-password -n argocd

# Login using the exiting port-forwarding
argocd login 127.0.0.1:8080

# list the applications
argocd app list

# Using kubectl command
k -n argocd get applications -o wide

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

## Reconciliation loop

By default ArgoCD have a reconciliation timeout to 180s

from values.yaml
```yaml
configs:
  cm:
    timeout.reconciliation: 180s
```

### Verify from an installed ArgoCD

```bash

# From the Helm chart

# From config map
k -n argocd get cm argocd-cm -o yaml
k -n argocd get cm argocd-cm -o jsonpath="{.data.timeout\.reconciliation}"

# From the pod running
k -n argocd get po my-argocd-repo-server-68d5f98f9c-72kgf -o yaml

```

## Update Configuration with Helm chart

Configuration changes:
* Reconciliation timeout to 1 day
* ArgoCD Server service from ClusterIP to NodePort 30085


1. Save the values of a Helm release to a YAML file, to read the entire values

```bash
# Save the values of a Helm release to a YAML file, to read the entire values
helm -n argocd get values my-argocd --all > argocd-custom-values.yaml
```

2. Remove all, and leave only the necessary ([./argocd-custom-values.yaml](./argocd-custom-values.yaml))

```yaml
configs:
  cm:
    timeout.reconciliation: 60s
server:
  service:
    nodePortHttp: 30085
    nodePortHttps: 30445
    type: NodePort
```

3. Upgrade the ArgoCD

```bash
helm -n argocd upgrade my-argocd argo/argo-cd -f argocd-custom-values.yaml
```

4. Verify

```bash
# Reconciliation timeout
k -n argocd get cm argocd-cm -o yaml

# NodePort ArgoCD server
k -n argocd get svc 

```

## Define a Custom Health Check in argocd-cm ConfigMap
Argo CD supports custom health checks written in Lua. This is useful if you:
* Are affected by known issues where your Ingress or StatefulSet resources are stuck in Progressing state because of bug in your resource controller.
* Have a custom resource for which Argo CD does not have a built-in health check.


https://argo-cd.readthedocs.io/en/stable/operator-manual/health/

patch.yaml
```yaml
  data:
    resource.customizations.health.ConfigMap: |
      hs = {}
      hs.status = "Healthy"
       if obj.data.PROPERTY == "ValuesX" then
          hs.status = "Degraded"
          hs.message = "Use any PROPERTY other than ValuesX "
       end
      return hs
```

patch
```bash
kubectl patch configmap argocd-cm -n argocd --patch-file patch.yaml
```

## Git Webhook Configuration
https://argo-cd.readthedocs.io/en/stable/operator-manual/webhook/

Create The WebHook In The Git Provider:
Navigate to the settings page where webhooks can be configured. The payload URL configured in the Git provider should use the `/api/webhook` endpoint of your Argo CD instance (e.g. https://argocd.example.com/api/webhook).

## Declarative Setup - Applications
https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#applications
The Application CRD is the Kubernetes resource object representing a deployed application instance in an environment. It is defined by two key pieces of information:

source reference to the desired state in Git (repository, revision, path, environment)
destination reference to the target cluster and namespace. For the cluster one of server or name can be used, but not both (which will result in an error). Under the hood when the server is missing, it is calculated based on the name and used for any operations.

Example: [./declarative-apps/mono-app/custom-nginx.yaml](./declarative-apps/mono-app/custom-nginx.yaml)

Deploy:
```bash
k apply -f ./declarative-apps/mono-app/custom-nginx.yaml
```