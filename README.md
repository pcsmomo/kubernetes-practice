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

### Create Configmap

[Kubernetes ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)

```sh
k apply -f mongo-configmap.yaml
# configmap/mongodb-configmap created
k apply -f mongo-express.yaml
# deployment.apps/mongo-express created

k get pods
# NAME                                  READY   STATUS    RESTARTS   AGE
# mongo-express-5bcd46fcff-dl6x6        1/1     Running   0          102s
# mongodb-deployment-5d966bd9d6-29g5w   1/1     Running   0          42m

k logs mongo-express-5bcd46fcff-dl6x6
# Welcome to mongo-express
# ------------------------


# (node:7) [MONGODB DRIVER] Warning: Current Server Discovery and Monitoring engine is deprecated, and will be removed in a future version. To use the new Server Discover and Monitoring engine, pass option { useUnifiedTopology: true } to the MongoClient constructor.
# Mongo Express server listening at http://0.0.0.0:8081
# Server is open to allow connections from anyone (0.0.0.0)
# basicAuth credentials are "admin:pass", it is recommended you change this in your config.js!
```

### Make it an External Service

- [Kubernetes Services - Type LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer)
- nodePort
  - Port for external IP address
  - Must be between 30000 - 32767

```sh
k apply -f mongo-express.yaml
# deployment.apps/mongo-express unchanged
# service/mongo-express-service created

k get services
# NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
# kubernetes              ClusterIP      10.96.0.1        <none>        443/TCP          4h7m
# mongo-express-service   LoadBalancer   10.104.125.184   <pending>     8081:30000/TCP   5s
# mongodb-service         ClusterIP      10.109.5.44      <none>        27017/TCP        68m

# mongo-express-service is pending
# because minikube will allocate it when service is open below
minikube service mongo-express-service
# |-----------|-----------------------|-------------|---------------------------|
# | NAMESPACE |         NAME          | TARGET PORT |            URL            |
# |-----------|-----------------------|-------------|---------------------------|
# | default   | mongo-express-service |        8081 | http://192.168.49.2:30000 |
# |-----------|-----------------------|-------------|---------------------------|
# 🏃  Starting tunnel for service mongo-express-service.
# |-----------|-----------------------|-------------|------------------------|
# | NAMESPACE |         NAME          | TARGET PORT |          URL           |
# |-----------|-----------------------|-------------|------------------------|
# | default   | mongo-express-service |             | http://127.0.0.1:51712 |
# |-----------|-----------------------|-------------|------------------------|
# 🎉  Opening service default/mongo-express-service in default browser...
# ❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.
```

Navigate `http://127.0.0.1:51712`

### Reset all

```sh
k delete -f mongo-express.yaml
k delete -f mongo.yaml
k delete -f mongo-configmap.yaml
k delete -f mongo-secret.yaml
```

## 9. Organizing your components with K8s Namespaces

```sh
k get namespaces
# NAME                   STATUS   AGE
# default                Active   34h
# kube-node-lease        Active   34h
# kube-public            Active   34h
# kube-system            Active   34h
# kubernetes-dashboard   Active   30h

kubens
# default
# kube-node-lease
# kube-public
# kube-system
```

- kube-system
  - Do NOT create or modify in kube-system
  - System processes
  - Master and Kubectl processes
- kube-public
  - publicely accesible data
  - A configmap, which contains cluster information
    ```sh
    kubectl cluster-info
    # Kubernetes control plane is running at https://127.0.0.1:59700
    # CoreDNS is running at https://127.0.0.1:59700/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
    ```
- kube-node-lease
  - hearbeats of nodes
  - each node has associated lease object in namespace
  - determins **the availability of a node**
- default
  - resources you create are located here
  - before you create new name spaces
    ```sh
    kubectl create namespace my-namespace
    kubectl get namespace
    ```
- kubernetes-dashboard

> However, it'd be better to create a namespace with a configuration file (`kind: ConfigMap`)\
> As it has history

### Why use namespaces?

1. Resources grouped in Namespaces
   - Database
   - Monitoring
   - Elastic Stack
   - Nginx-Ingress
2. Conflicts: Many teams, same application
   - specnetes
   - imagenates
   - scattering
   - and so on..
3. Resource Sharing
   - Staging and Development
   - Blue/Green Development
4. Access and Resource Limits on Namespace
   - Project A Namespace
     - Limit: CPU, RAM, Storage per namespace
   - Project B Namespace

### usecase

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-configmap
  namespace: my-namespace
data:
  db_url: mysql-service.database
```

### Components, which can't be created within a Namespace

- live globally in a cluster
- you can't isolate them

```sh
kubelctl api-resources --namespaced=false
kubelctl api-resources --namespaced=true
```

### Create component in a Namespace

```sh
kubectl create namespace my-namespace
k apply -f mysql-configmap.yaml
k get configmap
# NAME               DATA   AGE
# kube-root-ca.crt   1      47h
k get configmap -n default

k get configmap -n my-namespace
# NAME               DATA   AGE
# kube-root-ca.crt   1      61s
# mysql-configmap    1      50s

k get all -n my-namespace
# No resources found in my-namespace namespace.

# k apply -f mysql-configmap.yaml --namespace=my-namespace
```

### Change active namespace

kubens

```sh
brew install kubens
kubens
# default
# kube-node-lease
# kube-public
# kube-system
# kubernetes-dashboard
# my-namespace

# change the default namespace to my-namespace
kubens my-namespace
# Context "minikube" modified.
# Active namespace is "my-namespace".
```

### Clean up

```sh
k delete -f mysql-configmap.yaml
k delete namespace my-namespace
```

## 10. K8s Ingress explained

- [Ingress Controllser](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
- Ingress Controller Pod: Extra pod to handle
  - evaluates all the rules
  - manages redirections
  - endtrypoint to cluster
  - many third-party implementations

### Proxy server

- pointing to Ingress Controller Pod
- separate server
- public ID Address and open ports
- entrypoint to cluster

### Install Ingress

```sh
# No need to install manually anymore
minikube addons enable ingress
# 💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
# You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
# 💡  After the addon is enabled, please run "minikube tunnel" and your ingress resources would be available at "127.0.0.1"
#     ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20220916-gd32f8c343
#     ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20220916-gd32f8c343
#     ▪ Using image registry.k8s.io/ingress-nginx/controller:v1.5.1
# 🔎  Verifying ingress addon...
# 🌟  The 'ingress' addon is enabled

# Optional?
minikube tunnel
# ✅  Tunnel successfully started
# 📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...


k get pod -n kube-system
# ingress is not here.

k get ns
# NAME                   STATUS   AGE
# default                Active   3d
# ingress-nginx          Active   4m28s
# kube-node-lease        Active   3d
# kube-public            Active   3d
# kube-system            Active   3d
# kubernetes-dashboard   Active   2d20h

k get pod -n ingress-nginx
# NAME                                       READY   STATUS      RESTARTS   AGE
# ingress-nginx-admission-create-rhr7d       0/1     Completed   0          5m8s
# ingress-nginx-admission-patch-nrd6x        0/1     Completed   0          5m8s
# ingress-nginx-controller-77669ff58-4xvfr   1/1     Running     0          5m8s
```

### Create Ingress rule

Make `kubernetes-dashboard` accessible

```sh
k get all -n kubernetes-dashboard
# NAME                                            READY   STATUS    RESTARTS      AGE
# pod/dashboard-metrics-scraper-5c6664855-xmxdp   1/1     Running   1 (39h ago)   2d20h
# pod/kubernetes-dashboard-55c4cbbc7c-vstxh       1/1     Running   2 (39h ago)   2d20h

# NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
# service/dashboard-metrics-scraper   ClusterIP   10.101.19.251    <none>        8000/TCP   2d20h
# service/kubernetes-dashboard        ClusterIP   10.111.178.191   <none>        80/TCP     2d20h

# NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/dashboard-metrics-scraper   1/1     1            1           2d20h
# deployment.apps/kubernetes-dashboard        1/1     1            1           2d20h

# NAME                                                  DESIRED   CURRENT   READY   AGE
# replicaset.apps/dashboard-metrics-scraper-5c6664855   1         1         1       2d20h
# replicaset.apps/kubernetes-dashboard-55c4cbbc7c       1         1         1       2d20h
```

```sh
# kubernetes-nana/04-ingress
k apply -f dashboard-ingress.yaml
# ingress.networking.k8s.io/dashboard-ingress created

k get ingress -n kubernetes-dashboard
# NAME                CLASS    HOSTS           ADDRESS   PORTS   AGE
# dashboard-ingress   <none>   dashboard.com             80      43s
k get ingress -n kubernetes-dashboard --watch
# NAME                CLASS    HOSTS           ADDRESS        PORTS   AGE
# dashboard-ingress   <none>   dashboard.com   192.168.49.2   80      57s

sudo vim /etc/hosts
# 192.168.49.2 dashboard.com
```

Navigate `http://dashboard.com` -> Not working for me..

```sh
k describe ingress dashboard-ingress -n kubernetes-dashboard
# Name:             dashboard-ingress
# Labels:           <none>
# Namespace:        kubernetes-dashboard
# Address:          192.168.49.2
# Ingress Class:    <none>
# Default backend:  <default>
# Rules:
#   Host           Path  Backends
#   ----           ----  --------
#   dashboard.com
#                  /   kubernetes-dashboard:80 (10.244.0.17:9090)
# Annotations:     <none>
# Events:
#   Type    Reason  Age                From                      Message
#   ----    ------  ----               ----                      -------
#   Normal  Sync    97s (x3 over 11m)  nginx-ingress-controller  Scheduled for sync
```

> So with ingress, you can figure multiple services in the same domain\
> but subdomain by adding `- path` under `paths`\
> or different paths by adding new `- host: analytics.myapp.com` under `rules`

### Configuring TLS Certificate - https://

- [myapp-ingress-tls.yaml](./kubernetes-nana/04-ingress/myapp-ingress-tls.yaml)
- [myapp-secret.yaml](./kubernetes-nana/04-ingress/myapp-secret.yaml)

## 11. Helm - Package Manager

[Heml](https://helm.sh/docs/intro/install/)

- Package Manager for Kubernetes
- To package YAML files and distribute them in public and private repositories

### Helm Charts

- Bundle of YAML files
- Create your own Helm Charts with Helm
- Push them to Helm Repository
- Download and use existing ones

[Promotheus - an open-source systems monitoring](https://prometheus.io/docs/introduction/overview/)

### Sharing Helm Charts

```sh
helm search <keyword>
```

[Artifact Hub - Helm Chart Hub?](https://artifacthub.io/)

### Template Engine

1. Define a common blueprint
2. Dynamic values are replaced by placeholders

- [example.template.yaml](./kubernetes-nana/05-helm/example-template.yaml)
- [values.yaml](./kubernetes-nana/05-helm/values.yaml)

#### Usecase

- Same structure of cluster
  - mex, xas, Etc.
- Same application accross different environments
  - dev, stage, prod, Etc.

### Helm Chart Structure

```
mychart/
  Chart.yaml
  values.yaml
  charts/
  templates/
```

- Top level `mychart` folder: name of chart
- `Chart.yaml`: meta info about chart
- `values.yaml`: values for the template files
- `charts` folder: chart dependencies
- `templates` folder: the actual template files

```sh
heml install <chartname>
```

### value injection into template files

```sh
helm install --values=my-values.yaml <chartname>
# or
helm install --set version=2.0.0
```

### Release Management

- Helm v2
  - Heml CLI sends requests to `Tiller`
    `heml install <chartname>`
    `heml upgrade <chartname>`
    - changes are applied to existing deployment instead of creating a new one
    - Handling rollbacks
  - Downsides of Tiller
    - `Tiller` has too much power inside of K8s cluster
    - Security issue
- Helm v3
  - `Tiller` got removed

</details>
