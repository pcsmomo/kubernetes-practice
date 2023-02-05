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

</details>
