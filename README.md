# Kubernetes Practice

Kubernetes tutorials and practices related it

## Folder structure

-

# Details

<details open>
  <summary>Click to Contract/Expend</summary>

## 3. Main K8s Components

- Node
- Pod
- Service
- Ingress
  - contains services

### Environment

- ConfigMap
- Secret

### Data

- Volume

### Environment

- Deployment - blueprint for pods
- StatefulSet - for database state managing
  - it's challenging. having DB outside of the cluster is one way to go.

## 4. K8s Architecture

### 3 Node processes

- Kubelet - should be installed in every node
  - interacts with both the container and node
  - kubelet starts the pod with a container inside
- Kube proxy - should be installed in every node
- Container runtime

### Master Node Processes

- API server
  - cluster gateway
  - authentication
  - load balanced
- Scheduler
  - which pod should be scheduled next
- Controller manager
  - detect cluster state changes. e.g pod dies
- etdc: cluster brain
  - cluster changes get stored in the key value store

### 5. Minikube and kubectl - Local Setup

```sh
# brew install hyperkit
# hyperkit: The x86_64 architecture is required for this software.
# > just using docker

# brew install minikube
# I've already installed minikube from docker app

minikube status
# minikube
# type: Control Plane
# host: Running
# kubelet: Running
# apiserver: Running
# kubeconfig: Configured

minikube version
# minikube version: v1.29.0
# commit: ddac20b4b34a9c8c857fc602203b6ba2679794d3

# minikube start --vm-driver=hyperkit
minikube start --driver=docker

kubectl get nodes
# NAME       STATUS   ROLES           AGE     VERSION
# minikube   Ready    control-plane   4m27s   v1.26.1

kubectl version
kubectl version --short
# Client Version: v1.26.1
# Kustomize Version: v4.5.7
# Server Version: v1.26.1
```

</details>
