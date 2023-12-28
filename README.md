# Kubernetes

Watch: https://www.youtube.com/watch?v=dfxrdoEQe00&list=PLdpzxOOAlwvJdsW6A0jCz_3VaANuFMLpc

> [!INFO]
>💡 Kubernetes is Greek for **"helmsman of a ship”**

# A brief history of Kubernetes

Containers began as a Linux kernel process isolation.

- **namespaces** (2002) first allowed complete isolation of an application's view.
- **Borg** (2003) was a cluster manager by Google, that led to widespread use of similar approaches.
- **cgroups** (2007) first allowed limitation and priority of CPU, memory, etc.
- **LXC** (2008) is an OS-level virtualization for running multiple Linux systems on a single host.
- **Docker** (2013) completely popularized containers for the masses. At the time, **Mesos** was the primary tool for orchestrating containers. Kubernetes beat out **Docker Swarm** in the market.
- **Kubernetes** (2014) is a container orchestration tool, developed by Google as an open source version of Borg (2003).
    - Kubernetes was donated by Google to the **Cloud Native Computing Foundation**, a vendor-neutral home, in 2015.

After learning the fundamentals, there are **2 roles**, just like you learn a specific roadmap within AWS based on your role.

- The **Administrator** roadmap includes provisioning and configuring operations for usage.
    - Pros and Cons of EKS, AKS, GKE, DOMK
    - Network policies, pod-to-pod communication, cluster authentication, backups (By default, K8s has no data persistence!), Operators (no human intervention), monitoring/alerting (Promethes, Alertmanager, Grafana)
- The **Platform User** roadmap includes deploying and run applications within the cluster with high availability.
    - Deployment strategies (canary, blue-green, rolling updates/rollbacks), CI/CD, replicating your app and scaling up/down to adjust for load.

# Why Kubernetes?

- High **availability** or no downtime.
- **Scalability** when your application needs to scale up/down.
- **Disaster recovery**, backup and restore if the server explodes.
- Kubernetes is the future.

# How K8s works behind the scenes

- A **Control Plane** is a master node that creates a **virtual network** to turn worker nodes into one unified cluster, and runs the following master processes as **containers**, so it doesn’t need as many resources as worker nodes.
    - **Scheduler** (sched) **decides** on which Node the next new Pod should be scheduled and placed on depending on the available resources.
    - **Controller Manager** (c-m) keeps track of what’s happening in the cluster.
    - **etcd** is a K8s key-value store that contains the status of each Node. Backups are made from etcd snapshot.
    - **API server** (apiserver) is the ONLY entrypoint into K8s cluster for any UI, API, CLI.
- A **Node** is a worker node
    - **kubelet** is a node agent that makes it possible for the cluster to talk to each other.
    - kube-proxy
    - Container runtime

<aside>
💡 Always have **2 masters** in your cluster in production. If you lose a master node, you won’t be able to access the cluster.

</aside>

# Fundamental Concepts

The Kubernetes architecture is made up of at least one **master node** (**Control Plane**), with a couple of connected **worker nodes** that have a **kubelet** process running on it.

- A **Pod** is an **abstraction** over a container, so you don’t have to work directly with the Docker container runtime. The smallest unit in K8s, 1 application per pod.
    - Pods are **ephemeral**, and die when the 1) application inside crashes, 2) a new version of the application is available, or 3) the Node ran out of resources, a new one is created in its place with a new IP address.
- A **Service** (SVC) (eg. `postgres`) is a permanent static IP address with a DNS name for **communication** with Pods (like a load balancer for Pods). Can be external (port exposed) or internal.
    - **ClusterIP** (default) is only reachable within the cluster (exec in and curl)
    - **NodePort** exposes the IP outside the cluster
- A **ReplicaSet** dynamically drives the cluster back to the **desired state** via the creation of new, interchangable Pods to keep your application running.
- An **Ingress** sits in front of a service, and routes traffic into the cluster.
- **Config Map** (CM) is used to store **non-confidential** configuration data, like the URL of an external database, and is stored as plaintext.
- **Secret** is used to store **confidential** data, like **env variables**, and is stored as base64 encoded (not encrypted out-of-the-box, need 3rd party tool).
- **Volumes** attach a **persistent**, physical storage to your Pod, on the same worker node, or cloud storage. By default, K8s doesn’t manage data persistence.
- A **Deployment** is an **abstraction** over Pods, where you can define a blueprint for “my-app” Pod replicas.
    - You usually work with Deployments, not Pods, for statelets applications.
    - You shouldn’t create database Deployments, or you will have data inconsistencies
- A **StatefulSet** (STS) is a Deployment for stateFUL applications and databases, but it is tedious and challenging since you need to control when a database STS can read/write. It’s better to host databases outside the K8s cluster.

# K8s Configuration File

Requests to the entrypoint (apiserver) must be in YAML or JSON format. For example, configure a component called Deployment = a template for creating Pods. The **Controller Manager** checks desired state = actual state.

- A **configuration file** has 3 parts:
    - `metadata` (name) is what **kind** of blueprint you want to create.
    - `spec` contains **attributes** specific to the kind.
    - A hidden `status` is automatically generated and added by etcd
- It’s a good practice to store these with your code.

# How to access and interact with K8s

- **minikube** is an open source environment to run Kubernetes locally for day-to-day development.
    - **Master and Worker processes run on 1 Node**, with Docker pre-installed
    - minikube runs as a **Docker container**, with Docker pre-installed inside it. 🤪

```jsx
brew install minikube
minikube start

docker ps -a
// -> gcr.io/k8s-minikube/kicbase:v0.0.42
```

- **kubectl** (Kubernetes CLI) is a powerful CLI to communicate with the apiserver to create/destroy pods. Can be used for minikube cluster or a cloud cluster. Gets installed as a dep with minikube.
    - create
    - apply
    - get
    - describe
- K8s Manifest Files
- Helm Charts

# EKS

**Elastic Kubernetes Service** (EKS) is a Managed Kubernetes services that allows Amazon to manage Master Nodes, scaling and backups, and has all the necessary apps setup and configured for maximum security, so you can focus on deploying your applications.

- Warning: It’s a lot of effort compared to DigitalOcean, and you already have Kubernetes to learn. But it’s popular and powerful.
- Use **eksctl**, then go back and do it manually.
1. Signup for **AWS Free Tier**
2. Create a **VPC** (Private space)
3. Create an **IAM role** (Create AWS user) with a **Security Group** (with list of permissions)
4. Create **Cluster Control Plane** (2 Master Nodes), choose name, version, region, security.
5. Create **Node Group** (Worker Nodes on EC2 instances), choose cluster to attach to, security group, instance type, resources. Define max and min number of nodes.
6. Connect to the cluster from your local machine using **kubectl.**

```jsx
brew install eksctl awscli
awscli configure
// Manually create an IAM with full Admin privileges

eksctl create cluster \
--name test-cluster \
--nodegroup-name linux-nodes \
--node-type t2.micro \
--nodes 2

kubectl get nodes
```

- Takes aprox. 15 minutes to build the cluster using CloudFormation and creates ~/.kube/config with the cluster IP.

```jsx
eksctl delete cluster --name test-cluster
```

- Terminates then deletes the resources (takes some time)

# Demo

```jsx
kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
```

- Searches for a suitable node (we only have 1), schedules the application to run, and configures the cluster to reschedule the instance when needed.

# Scaling

<aside>
💡 Scaling to **0** is also possible, which will terminate all Pods in the Deployment.
</aside>

# Common K8s Misconfigurations

K8s is not very opinionated, so it’s best to learn the DON’Ts before the DO’s.

<aside>
💡 If you're running minikube with **Docker Desktop** as the container driver, a minikube tunnel is needed. This is because containers inside Docker Desktop are isolated from your host computer.

</aside>

- minikube service kubernetes-bootcamp --url

# Infrastructure as YAML, not code

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: site
  labels:
    app: web
spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
```

- **Kubernetes YAML files** allows DevOps to express their deployment without the need to write code in a programming language.
- Unlike symlinked nginx configs in a sites-enabled folder, YAML can be easily version controlled.
- Having resources defined as YAML makes it easy to scale by changing one or two numbers.
- Policy validators, such as **rego**, can check that your workloads don't allow a container to run as root.

```yaml
package main

deny[msg] {
  input.kind = "Deployment"
  not input.spec.template.spec.securityContext.runAsNonRoot = true
  msg = "Containers must not run as root"
}
```

- Seamless integration with cloud providers since 2018 (Digital Ocean)
- Extensibility. We can create a CronTab **resource** with something like this:

```yaml
apiVersion: "my.org/v1"
kind: CronTab
metadata:
  name: my-cron-object
spec:
  cronSpec: "* * * * */5"
  image: my-cron-image
  replicas: 5
```

# Innovation

Kubernetes has had major releases every 3-4 months over the last few years. 

- There are a wide range of special interest groups (SIG) that target different areas of Kubernetes.

# Launching a Kubernetes Cluster

1. Start cluster with minikube
2. Launch deploments, replication controllers, and expose them via Services with kubectl/yaml definitions

**Minikube** runs Kubernetes locally for day-to-day development. 

- To start a cluster run **minikube start**

```yaml
$ minikube start --wait=false
```

The cluster can then be interacted with using the **kubectl** CLI.

- To view its health run **kubectl cluster-info**
- To view the nodes in the cluster run **kubectl get nodes**

```yaml
NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   4m17s   v1.17.3
```

- If the node is marked as NotReady then it is still starting the components.

With a running cluster, containers can now be deployed using **kubectl run**

```bash
# kubectl run <name of deployment> <image and replicas>
$ kubectl run http --image=katacoda/docker-http-server:latest --replicas=1

# View the status of deployments
$ kubectl get deployments

# Describe the deployment process
$ kubectl describe deployment http

# Get services
$ kubectl get svc
$ docker pos | grep httpexposed
```

- The description includes how many replicas are available, labels specified and the events associated with the deployment. These events will highlight any problems and errors that might have occurred.
- The run command creates a deployment based on the parameters specified, such as the image or replicas
- Similar to docker run but at a cluster level.

```yaml
$ kubectl create deployment first-deployment --image=katacoda/docker-http-server
```

- To view the running pods run **kubectl get pods**

```yaml
NAME                               READY   STATUS    RESTARTS   AGE
first-deployment-666c48b44-gmtvr   1/1     Running   0          7m26s
```

Once the container is running, it can be exposed via different networking options.

- Under the hood, this exposes the Pod via Docker Port Mapping.

# Minikube dashboard

You can enable the dashboard using Minikube.

```yaml
$ minikube addons enable dashboard
```

# Pause contrainers

You'll notice the prots are exposed on the Pod, not the containers themselves. The **Pause container** is reponsible for defining the network for the pod. This improves network performance.

# Scaling Containers

**Rolling updates** allow Deployments' update to take place, CI/CD with zero downtime by incrementally updating Pods instances with new ones.

Scaling the deployment will request Kubernetes to launch additional **Pods**. 

- You should see three running for the http deployment

```bash
# kubectl scale <replicas> deployment <name>
$ kubectl scale --replicas=3 deployment http

$ kubectl get pods
```

These Pods will then automatically be load balanced using the exposed **Service**.

- You should see three endpoints and the associated pod

```bash
$ kubectl describe svc http
```

Making requests to the service will request in different nodes processing the request.

```bash
$ curl http://172.17.0.56:8000 

# http-774bb756bb-8j7zc
# http-774bb756bb-n5hgr
# http-774bb756bb-8j7zc
# http-774bb756bb-ml6zh
```

# 1. Deploying containers using YAML

The **deployment object** is one of the most common Kubernetes objects.

- It defines the container spec, name, and labels used by other parts of Kubernetes
- The following definition defines how to launch an application called "webapp1 using the docker image "katacode/docker-http-server" that runs on port 80

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp1
  template:
    metadata:
      labels:
        app: webapp1
    spec:
      containers:
      - name: webapp1
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
```

This is deployed to the cluster with the command

```yaml
$ kubectl create -f deployment.yaml
# deployment.apps/webapp1 created
```

And it's a Deployment object, a list of all the deployed objects, can be obtained via

```yaml
$ kubectl get deployment
# NAME      READY   UP-TO-DATE   AVAILABLE   AGE
# webapp1   1/1     1            1
```

Details of the individual deployments can be outputted with 

```yaml
$ kubectl describe deployment webapp1
# Name:                   webapp1
# Namespace:              default
# CreationTimestamp:      Sat, 06 Jun 2020 19:45:02 +0000
```

# 2. Create Service

The Service selects all applications with the label webapp1. As multiple replicas, or instances, are deployed, they will be automatically load balanced based on this common label. The Service makes the application available via a **NodePort**.

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp1-svc
  labels:
    app: webapp1
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30080 # <--- NodePort
  selector:
    app: webapp1
```

Deploy the service

```yaml
$ kubectl create -f service.yaml
# service/webapp1-svc created
```

As before, get details of all the Service objects

```yaml
$ kubectl get svc
# NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
# kubernetes    ClusterIP   10.96.0.1      <none>        443/TCP        19m
# webapp1-svc   NodePort    10.97.31.237   <none>        80:30080/TCP   74s
```

```yaml
$ curl host01:30080
# <h1>This request was processed by host: webapp1-6b54fb89d9-mwl49</h1>
```

# 3. Scale deployment

Update the deployment.yaml file to increase the number of instances running using **kubectl apply**.

```yaml
$ kubectl apply -f deployment.yaml
```

As all Pods have the same label selector, they'll be load balanced behind the Service NodePort deployed.

```yaml
$ kubectl get pods
```

# Example gusetbook

We'll be storing notes in Redis via JavaScript API calls. Redis contains a master (for storage) and a replicated set of redis 'slaves'.
