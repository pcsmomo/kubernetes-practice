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

### 6. Main Kubectl Commands - K8s CLI

```sh
alias k
# k=kubectl
k get pods
k get services
# NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
# kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   27m

k create -h
k create deployment nginx-depl --image-nginx
# deployment.apps/nginx-depl created
k get deployment
# NAME         READY   UP-TO-DATE   AVAILABLE   AGE
# nginx-depl   0/1     1            0           26s
k get pods
# NAME                         READY   STATUS    RESTARTS   AGE
# nginx-depl-56cb8b6d7-n9s86   1/1     Running   0          63s
```

> we'd better not control pod itself, but deployment

```sh
k get replicaset
NAME                   DESIRED   CURRENT   READY   AGE
nginx-depl-56cb8b6d7   1         1         1       117s
```

> Replicaset is managin the replicas of a poda

```sh
# change nginx image version
# image: nginx -> image: nginx:1.16
k edit deployment nginx-depl
# deployment.apps/nginx-depl edited
k get pods
# NAME                          READY   STATUS        RESTARTS   AGE
# nginx-depl-56cb8b6d7-n9s86    1/1     Terminating   0          5m32s
# nginx-depl-8475696677-ddbnr   1/1     Running       0          50s
```

#### Debugging pods

```sh
k logs nginx-depl-8475696677-ddbnr
k create deployment mongo-depl --image=mongo
k get pods
# NAME                          READY   STATUS              RESTARTS   AGE
# mongo-depl-5ccf565747-jqgrn   0/1     ContainerCreating   0          11s
# nginx-depl-8475696677-ddbnr   1/1     Running             0          3m21s
k logs mongo-depl-5ccf565747-jqgrn
# Error from server (BadRequest): container "mongo" in pod "mongo-depl-5ccf565747-jqgrn" is waiting to start: ContainerCreating
k describe pod mongo-depl-5ccf565747-jqgrn
# Name:             mongo-depl-5ccf565747-jqgrn
# Namespace:        default
# Priority:         0
# Service Account:  default
# Node:             minikube/192.168.49.2
# ...
k logs mongo-depl-5ccf565747-jqgrn
# {"t":{"$date":"2023-02-06T21:51:20.673+00:00"},"s":"I",  "c":"CONTROL",  "id":23285,   "ctx":"main","msg":"Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'"}
# {"t":{"$date":"2023-02-06T21:51:20.673+00:00"},"s":"I",  "c":"NETWORK",  "id":4915701, "ctx":"main","msg":"Initialized wire specification","attr":{"spec":{"incomingExternalClient":{"minWireVersion":0,"maxWireVersion":17},"incomingInternalClient":{"minWireVersion":0,"maxWireVersion":17},"outgoing":{"minWireVersion":6,"maxWireVersion":17},"isInternalClient":true}}}
k exec -it mongo-depl-5ccf565747-jqgrn -- bin/bash
root@mongo-depl-5ccf565747-jqgrn:/# ls
```

#### Delete pods

```sh
k get deployment
# NAME         READY   UP-TO-DATE   AVAILABLE   AGE
# mongo-depl   1/1     1            1           5m
# nginx-depl   1/1     1            1           12m
k delete deployment mongo-depl
# deployment.apps "mongo-depl" deleted
k get pods
# NAME                          READY   STATUS    RESTARTS   AGE
# nginx-depl-8475696677-ddbnr   1/1     Running   0          9m9s
k get replicaset
k delete deployment nginx-depl
```

#### Custom deployment file

```sh
# ./kubernetes-nana
touch 01-nginx-deployment.yaml
k apply -f 01-nginx-deployment.yaml
# deployment.apps/nginx-deployment created
k get pods
# NAME                               READY   STATUS    RESTARTS   AGE
# nginx-deployment-95585b474-z8ljx   1/1     Running   0          33s

# change replicas to 2
apply -f 01-nginx-deployment.yaml
# deployment.apps/nginx-deployment configured
k get pods
# NAME                               READY   STATUS    RESTARTS   AGE
# nginx-deployment-95585b474-88nnp   1/1     Running   0          8s
# nginx-deployment-95585b474-z8ljx   1/1     Running   0          61s
```

</details>
