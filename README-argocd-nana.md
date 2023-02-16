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

</details>
