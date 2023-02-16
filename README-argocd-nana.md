# ArgoCD Tutorial

ArgoCD tutorials and practices related it

## Folder structure

-

# Details

<details open>
  <summary>Click to Contract/Expend</summary>

## 1. Intro and Overview

- What is ArgoCD?
- Why ArgoCD? Its use cases
- How ArgoCD works & benefits?
- Hands-on Demo Project:
  - Set up a fully automated CD pipeline

## 2. What is ArgoCD

Continuous Delivery Tool

### How ArgoCD compares to them?

#### CD workflow without ArgoCD, such as Jenkins

- CI
  - test
  - build image
  - push to docker repo
- CD
  - update k8s manifest file
  - `kubectl apply ...`
  - new version deployed!

This is the way many applications are set up.\
However, challenges with this approach:

- 🚫 Install and setup tools, like kubectl
- 🚫 Configure access to K8s
- 🚫 configure access to cloud platforms
- 🚫 Security challenge
- 🚫 **No visibility of deployment status after sending**

#### ArgoCD is...

- CD for Kubernetes
- Based on GitOps

#### ArgoCD as a better alternative

Reverse the flow

- ArgoCD is part of K8s cluster
- ArgoCD agents pulls K8s manifest(k8s, yaml files) changes and applies them

#### CD workflow with ArgoCD

1. Deploy ArgoCD in K8s cluster
2. Configure ArgoCD to track Git repository
3. ArgoCD monitors for any changes and applies automatically

### Best practice for git repository

- Separate git repository for **application source code** and **application configuration** (k8s manifest files)
- Even separate git repository for **system configurations**

### ArgoCD supports:

- Kubernetes YAML files
- Helm Charts
- Kustomize

## 5. Benefits of using GitOps with ArgoCD

- Git as Single Source of Truth
- Easy Rollback
- Cluster Disaster Recovery

## 10. ArgoCD as Kubernetes Extension

- ArgoCD uses existing K8s functionalities
  - E.G. using etcd to store data
  - E.G. using K8s controllers for monitoring and comparing actual and desired state

## 11. How to configure ArgoCD?

1. Deploy ArgoCD into K8s cluster
   - extends the K8s API with Custom Resource Definition (crd)
2. Configure ArgoCD with K8s YAML file
   - main resource is `kind: Application`
   - source : git repository
   - destination: k8s cluster

## 12. Multiple Clusters with ArgoCD

1. git branch for each environment
2. using overlays with kustomize
   - ./myapp-cluster
     - base
       - deployment.yaml
       - kustomization.yaml
       - rbac.yaml
       - service.yaml
     - overlays
       - development
         - kustomization.yaml
       - staging
         - kustomization.yaml
       - production
         - kustomization.yaml

## 14. Demo Setup & Overview

1. Install ArgoCD in K8s cluster
2. Configure ArgoCD with "Application" CRD
3. Test our setup by updating `Deployment.yaml` file

```sh
minikube status
```

- base image: https://hub.docker.com/r/nanajanashia/argocd-app/tags
- git lab repo: https://gitlab.com/nanuchi/argocd-app-config
- We are going to start from tag 1.0 and will update the tag versions

## 15. Beginning of Hands-On Demo

### Install ArgoCD

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/core-install.yaml
```

### Access Argocd UI

```sh
kubectl get svc -n argocd
kubectl port-forward -n argocd svc/argocd-server 8080:443
# Forwarding from 127.0.0.1:8080 -> 8080
# Forwarding from [::1]:8080 -> 8080
```

Navigate the Argocd Dashboard, https://127.0.0.1:8080

- id: admin
- password
  - [Login Using the CLI](https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli)
  - Or follow the lecture
    ```sh
    kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
    echo <password> | base64 --decode
    ```

### Set up the application config

[Argocd Declarative Setup](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#applications)

- ```yaml
  source:
    repoURL: https://gitlab.com/nanuchi/argocd-app-config.git
    targetRevision: HEAD
    path: dev
  ```
- `kubectl get svc`
  ```yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp
  ```
- if `myapp` namespace doesn't exist, create one
  ```yaml
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
  ```
- [ArgoCD Automated sync policy](https://argo-cd.readthedocs.io/en/stable/user-guide/auto_sync/#automated-sync-policy)
  ```yaml
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
  ```
  - Why are these turned off by default?
    - Safety Mechanism

#### Polling policy

- As default, ArgoCD polls Git repository every 3 minutes
- If you don't want this delay, you can configure a Git webghook

### Apply configuration

Push all changes to the repo that we configured

```sh
kubectl apply -f application.yaml
```

### Check the synced Git projec5

- Navigate the Argocd Dashboard, https://127.0.0.1:8080
- myapp-argo-application
  - summary tab and manifest tab

### Test Auto Sync

- Change the image tag number in `dev/deployment.yaml` from 1.0 to 1.2 on the git repo
  - ArgoCD will automatically notice the changes and apply it to the kubernetes cluster
- Change metadata name from `myapp-deployment` to `myapp` in `dev/deployment.yaml`

### Change the cluster directly

```sh
kubectl edit deployment -n myapp myapp
# change replicas from 2 to 4
```

- 2 more pods will be created but soon ArgoCD will remove 2 pods \
  as the replicas was set to 2 in the git repo

### Manually Sync

1. Refresh: Compare the latest code in Git with the live state
2. Sync: Move target state, by actually applying the changes to the K8s cluster

</details>
