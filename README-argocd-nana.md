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

</details>
