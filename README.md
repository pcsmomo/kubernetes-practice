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
- etcd: cluster brain
  - cluster changes get stored in the key value store

## 5. Minikube and kubectl - Local Setup

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

## 6. Main Kubectl Commands - K8s CLI

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

### Debugging pods

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

### Delete pods

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

### Custom deployment file

```sh
# ./kubernetes-nana/01-nginx

touch nginx-deployment.yaml
k apply -f nginx-deployment.yaml
# deployment.apps/nginx-deployment created
k get pods
# NAME                               READY   STATUS    RESTARTS   AGE
# nginx-deployment-95585b474-z8ljx   1/1     Running   0          33s

# change replicas to 2
apply -f nginx-deployment.yaml
# deployment.apps/nginx-deployment configured
k get pods
# NAME                               READY   STATUS    RESTARTS   AGE
# nginx-deployment-95585b474-88nnp   1/1     Running   0          8s
# nginx-deployment-95585b474-z8ljx   1/1     Running   0          61s
```

## 7. K8s YAML Configuration File

### Each configuration file has 3 parts

1. Metadata
2. Specification
   - attributes of "spec" are specific to the "kind"!
   - `kind: Deployment`
3. Status
   - always watching `desired (defined in spec) = actual`
   - where does k8s get this status data? `etcd`

### template

configuration for pods inside configuration for deployment

- `template: metadata: labels`
  - it's used for spec
    - `spec: selector: matchLabel`
  - it's also used for services
    - `service.yaml: spec: selector`

### port

`deployment.yaml: containerPort` = `service.yaml: targetPort`

```sh
# ./kubernetes-nana/01-nginx

k apply -f nginx-deployment.yaml
k apply -f nginx-service.yaml
# NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
# kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP   115m
# nginx-service   ClusterIP   10.102.253.136   <none>        80/TCP    6s
k describe service nginx-service
# Name:              nginx-service
# Namespace:         default
# Labels:            <none>
# Annotations:       <none>
# Selector:          app=nginx
# Type:              ClusterIP
# IP Family Policy:  SingleStack
# IP Families:       IPv4
# IP:                10.102.253.136
# IPs:               10.102.253.136
# Port:              <unset>  80/TCP
# TargetPort:        8080/TCP
# Endpoints:         10.244.0.6:8080,10.244.0.7:8080
# Session Affinity:  None
# Events:            <none>
k get pod -o wide
# NAME                               READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
# nginx-deployment-95585b474-88nnp   1/1     Running   0          49m   10.244.0.7   minikube   <none>           <none>
# nginx-deployment-95585b474-z8ljx   1/1     Running   0          50m   10.244.0.6   minikube   <none>           <none>
```

### Check the result of deployment (useful for debugging)

```sh
# check if the status has changed
k get deployment nginx-deployment -o yaml
k get deployment nginx-deployment -o yaml > nginx-deployment-result.yaml
```

> Tip❕\
> If you want to copy from the automated yaml file, you will have to remove most of generated stuff and create from it

```sh
# ./kubernetes-nana/01-nginx

k delete -f nginx-deployment.yaml
k delete -f nginx-service.yaml
```

## 8. Demo Project: MongoDB and MongoExpress

```sh
k get all
# NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
# service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   153m
```

### Create Secret

- [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/#use-cases)
- [Kubernetes Secrets - Types of Secret](https://kubernetes.io/docs/concepts/configuration/secret/#secret-types)

1. Generate encoded strings

```sh
# generate encoded string
echo -n 'username' | base64
# dXNlcm5hbWU=
echo -n 'password' | base64
# cGFzc3dvcmQ=
```

2. Apply secret
   > Run secret before using deployment or any others

```sh
k apply -f mongo-secret.yaml
k get secret
# NAME             TYPE     DATA   AGE
# mongodb-secret   Opaque   2      18s
```

3. Apply deployment

```sh
k apply -f mongo.yaml
k get all
# NAME                                      READY   STATUS              RESTARTS   AGE
# pod/mongodb-deployment-5d966bd9d6-29g5w   0/1     ContainerCreating   0          4s

# NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
# service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   172m

# NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/mongodb-deployment   0/1     1            0           4s

# NAME                                            DESIRED   CURRENT   READY   AGE
# replicaset.apps/mongodb-deployment-5d966bd9d6   1         1         0       4s

k get pod --watch
# NAME                                  READY   STATUS    RESTARTS   AGE
# mongodb-deployment-5d966bd9d6-29g5w   1/1     Running   0          27s

k describe pod mongodb-deployment-5d966bd9d6-29g5w
```

### Create Service

```sh
k apply -f mongo.yaml
# deployment.apps/mongodb-deployment unchanged
# service/mongodb-service created

# Check them out
k get service
# NAME              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)     AGE
# kubernetes        ClusterIP   10.96.0.1     <none>        443/TCP     179m
# mongodb-service   ClusterIP   10.109.5.44   <none>        27017/TCP   23s

k describe service mongodb-service
# Endpoints:         10.244.0.8:27017
k get pod -o wide
# NAME                                  READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
# mongodb-deployment-5d966bd9d6-29g5w   1/1     Running   0          7m42s   10.244.0.8   minikube   <none>           <none>
k get all
k get all | grep mongodb
# pod/mongodb-deployment-5d966bd9d6-29g5w   1/1     Running   0          8m26s
# service/mongodb-service   ClusterIP   10.109.5.44   <none>        27017/TCP   118s
# deployment.apps/mongodb-deployment   1/1     1            1           8m26s
# replicaset.apps/mongodb-deployment-5d966bd9d6   1         1         1       8m26s
```

### Mongo-express Deployment

[mongo-express docker hub](https://hub.docker.com/_/mongo-express)

</details>
