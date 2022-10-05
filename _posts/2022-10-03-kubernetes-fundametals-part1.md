---
title: "Kubernetes Fundamentals - Part 1"
date: 2022-10-03
classes: wide
categories:
  - Kubernetes
tags:
  - kubernetes
  - containers
---

Recently, I decided to brush up my Kubernetes skills and best way to do that is to write blog posts and share my notes with the community. This is a 4 part series where I explain various concepts of Kubernetes at a high level.

1. Part 1
2. Part 2
3. Part 3
4. Part 4

**Who is the target audience?**  
The target audience are the people like me who want to brush up their Kubernetes knowledge or people who are newly starting with Kubernetes and want to build a basic understanding of Kubernetes.

**Any pre-requisites?**  
The reader is expected to have basic understanding of Docker, Containerization and Container Registries.

**Does the articles include hands on labs or samples?**  
These articles will focus on the theoretical concepts of Kubernetes, hence there are no hands on labs included with these articles. However, I have provided links to Kubernetes official documentation where you can find good hands on examples about the concepts discussed in these articles.

**Which topics are discussed in part 1?**

1. [Kubernetes Architecture](#kubernetes-architecture)
2. [Pods](#pods)
3. [ReplicaSet](#replication-controller-or-replicaset)
4. [Deployment](#deployments)
5. [Services](#services)
6. [Namespace](#namespace)

## **Kubernetes Architecture**

![Kubernetes Architecture Diagram](/assets/images/kubernetes-fundametals/Kubernetes_architecture.svg)

- Nodes are physical or virtual machines that constitute a Kubernetes cluster. There are two types of nodes in a cluster:
  - Master Nodes - Responsible for managing the cluster
  - Worker Nodes - Used for running the workloads
- API Server - Acts as a frontend for Kubernetes. Users, command line, deployments etc. is done through the API Server
- etcd - Reliable distributed key-value data store. It is used for storing the data used by Kubernetes to manage the cluster. It is like a database for Kubernetes.
- Scheduler - Responsible for distributing work or containers across multiple nodes.
- Controller - Brain behind the orchestration. Node failures, container failover etc. are handled by controllers. E.g. Replication Controller, Endpoints Controller, Namespace Controller, ServiceAccount Controller etc.
- Container Runtime - Underlying software which is used to run containers like Docker, Moby, rkt, CRI-O etc.
- Kubelet [Only on worker node] - Agent that runs on each node and is responsible for interacting with API server. It also makes sure that the containers are running on the node as expected.
- Kube-proxy - kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept. kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.
- kubectl - Command line utility for deploying and managing applications on Kubernetes cluster.

## **Pods**

- Pod is the smallest object that can be created in Kubernetes. A pod is a single instance of an application.
- Containers are encapsulated within a Pod. Single Pod can have multiple containers.
- Containers within the same pod share the underlying resources like network, storage, state, CPU, Memory etc. Containers within the same pod can reach each other using localhost.
- Scaling in Kubernetes is done at the Pod level and not at the container level.
- Kubernetes always restarts the pod after it exits and this happens due to the "restartPolicy" property. By default the restart policy is set to "Always" which means even if the pod exits due to completion of its task, k8s still attempts to restart that pod. To overcome this you can set the `restartPolicy` to `Never` or `OnFailure`.

## **Replication Controller** or **ReplicaSet**

- Replication Controller helps us to ensure that a desired number of instances of a pod are always running (even if the desired number is 1). This allows us to ensure high availability for our application.
- Another reason we need replication controller is to create multiple pods to share the user load. As the demand grows we can increase the replica count and scale our application.
- Replication Controller (RC) is the older technology and is replaced by ReplicaSets now:
  - Replication Controller - apiVersion: v1, ReplicaSet - apiVersion: apps/v1
  - Another difference between the RC and ReplicaSet is, a ReplicaSet requires a "selector" parameter to be set in YAML. This parameter informs the ReplicaSet which set of Pods to monitor.
- ReplicaSet can also be created to monitor existing pods [But you still need to provide the YAML definition of existing pods]

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replica count according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
        - name: php-redis
          image: gcr.io/google_samples/gb-frontend:v3
```

## **Deployments**

- Deployment is the preferred way of deploying applications to Kubernetes. Deployments offer features like Rolling updates, Rollback to previous version etc.
- With deployment you can make multiple changes to the environment in one go (like scale the number of replicas, change container image version, change container resource allocation etc.)
- From YAML definition perspective the only diff. between `Deployment` and `ReplicaSet` is the _kind_ parameter which needs to be set to "Deployment" instead of "ReplicaSet". Rest of the properties are same as ReplicaSet definition.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

### Deployment Rollout and Versioning

- When you create a new deployment it creates a _Rollout_ which creates a _Revision_.
- Everytime you update your application through a deployment a new _Revision_ gets created. This helps Kubernetes keep track of the changes made to a deployment and enables us to Rollback to previous version.

```sh
# View deployment status
kubectl rollout status deployment/my-dep

# View deployment history
kubectl rollout history deployment/my-dep

# Rollback or undo a deployment
kubectl rollout undo deployment/my-dep
```

### Deployment Strategies

There are two types of deployment strategies supported by Kubernetes:

- **Recreate Strategy** - Destroy all existing instances and bring up the new version of the application [This will result in downtime]
- **Rolling update** (default strategy) - The existing pods are brought down one by one and newer version is deployed at the same time. [No application downtime]

_How does Rolling update strategy works?_
When you deploy a newer version of your application, Kubernetes creates a new `ReplicaSet` and starts deploying the new pods in that ReplicaSet. At the same time as new Pods are getting deployed one by one, Kubernetes also brings down the older pods in the old ReplicaSet. Once the old ReplicaSet is empty, k8s deletes that ReplicaSet. The Rollback features also works in the same but in reverse.  
Below is a screenshot describing how rolling update brings down each Replica one at a time and deploys the new replica. Here the replica count is 5:

<img src="/assets/images/kubernetes-fundametals/RollingUpdate.png" style="max-width: 1000px;">

<br/>
Below screenshot shows how the Recreate strategy brings down all the replicas at once and then deploys the new replica.
<img src="/assets/images/kubernetes-fundametals/Recreate.png" style="max-width: 900px;">

### Max Unavailable and Max Surge in a Rolling update strategy

- Max Unavailable - Deployment ensures that only a certain number of Pods are down while they are being updated. By default, it ensures that at least 75% of the desired number of Pods are up (25% max unavailable).
- Max Surge - Deployment also ensures that only a certain number of Pods are created above the desired number of Pods. By default, it ensures that at most 125% of the desired number of Pods are up (25% max surge).

For example, if you observe a Deployment closely, you will see that it first creates a new Pod, then deletes an old Pod, and creates another new one. It does not kill old Pods until a sufficient number of new Pods have come up, and does not create new Pods until a sufficient number of old Pods have been killed. For a deployment with 3 replicas, it makes sure that at least 2 Pods are available and that at max 4 Pods in total are available. In case of a Deployment with 4 replicas, the number of Pods would be between 3 and 5.

### Blue/Green and Canary Deployment Strategy

Apart from the Recreate and Rolling update deployment strategies you can also follow other deployment strategies like Blue/Green deployment, Canary deployment for your applications. These strategies are not something that you can configure in the deployment definition file but it is a way of releasing your application.

**Blue/Green Deployment**: In this strategy a new version of the application is deployed along with the old version. The old version is call _Blue_ and new version is called _Green_. Once the new version is deployed, 100% of the traffic is still routed to the Blue version while the tests are being run on the Green version. Once all tests are passed, traffic is switched to Green version all at once.

**Canary Deployment**: In this strategy we deploy a new version of the application along with the old version but route a small percentage of traffic is routed to the new version. Once all the tests are run successfully we upgrade the older version with the newer version of the application (which can be done by Rolling update strategy, for example). Finally, we remove the canary deployment which was deployed for running the tests.

These deployment strategies are best implemented with service meshes like Istio.

## **Services**

Kubernetes assigns an IP address to every Pod and the containers inside the Pod share that IP address. However, pods are regularly created and destroyed, causing their IP addresses to change constantly. This will result in discoverability issues as it will be difficult for the applications to connect with a pod if its IP address keeps on changing. So how do we manage communication with pods? How do we manage traffic routing or load balancing?

This is where the concept of Kubernetes Service comes into picture. Kubernetes services enable communication between various components both inside and outside of the Kubernetes Cluster. Service keep track of the changes in IP addresses and DNS names of the pods and expose them to the end-user as a single IP or DNS. It provides:

- Stable IP address
- Traffic routing to the Pods
- Load balancing
- Loose coupling
- Internal or external communication

A Service is not a Pod, it is an abstraction layer that represents an IP address.

There are three types of services in Kubernetes:

1. NodePort
2. ClusterIP
3. LoadBalancer

### NodePort

A NodePort Services makes a pod accessible over a particular port on the node. There are three ports involved when creating a `NodePort` service, these ports are from the service view point:

1. TargetPort: This is the Port at which the Pod is exposing the application. If we don't provide a target port, it is assumed to be same as the "Port".
2. Port: This is the port to be opened on the service itself (Service is like a Virtual Server inside the node. It has its own IP address known as the ClusterIP). This is a required property.
3. NodePort: Port to be opened on the Node. If we don't provide the NodePort then a free port in the valid range (Range 30000-32767) is allocated as NodePort.

<p align="center">
  <img src="/assets/images/kubernetes-fundametals/nodeport.svg" style="max-height: 600px;">
</p>

NodePort services can automatically distribute traffic across multiple Pods (It acts as a built-in LoadBalancer). It also works even if the pods are distributed across multiple nodes in the cluster. Kubernetes automatically creates a NodePort service spanning all the nodes in the cluster. You can use IP address and port on any node to connect to the Pod:

<p align="center">
  <img src="/assets/images/kubernetes-fundametals/nodeport_multinode.svg" style="max-height: 600px;">
</p>

### ClusterIP

ClusterIP service type exposes the services on an internal IP address. It makes the Service only reachable from within the cluster. This is the default service type. It is also known as _Internal Service_.

<img src="/assets/images/kubernetes-fundametals/ClusterIPService.png"  style="max-width: 600px;">

### LoadBalancer

_LoadBalancer_ service type creates a LB (with Public or Private IP address) in the supported Cloud Provider. This LB service is the standard way to expose a service to the internet.

### References

- [Ref. link with good comparison between different types of services](https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0)
- [Kubernetes documentation - Service](#https://kubernetes.io/docs/concepts/services-networking/service/)

## **Namespace**

- Namespace is like a virtual sub cluster inside a Kubernetes cluster. You can have multiple namespaces in a Kubernetes cluster and they are isolated from each other.
- Namespaces can help you with Organization, Security and Performance of your containerized workloads.
- You can also specify resource limitations at the namespace level.
- When you create a new Kubernetes cluster, by default there are following namespaces created:

  - default - Empty namespace which can be used by you for deploying your own applications.
  - kube-system - This namespace contains the components that are used for managing the system processes and you should not make any changes to this namespace.
  - kube-public - This namespace contains publicly accessible data. It contains a configmap which contains cluster information which is accessible without authentication. (This is the information which you get when you type 'kubectl cluster-info')
  - kube-node-lease - This is a recent addition to Kubernetes. This namespace holds information about the heartbeats of the nodes. Each nodes has an associated lease object in this namespace that contains information about that node's availability.

- Use cases:

  - If everything is in a single namespace, eventually it becomes difficult to manage. Logically grouping the resources inside the cluster provides a better way of managing the resources.
  - When you have multiple teams working on the same cluster. Each team can work with their own namespace without disrupting the other team.
  - Resource sharing through namespaces: Same cluster can be used for different environments like Development and Staging.
  - Blue/Green deployment for the application.
  - Limit the resource usage by an application at the namespace level (Limit: CPU, RAM, Storage). This helps in maintaining performance of an application since pods in one namespace cannot use the resources allocated to other namespace.
  - Limit the access to an applications from different teams. If every application is isolated in its own namespace we can apply network policies to control the traffic flow between pods in different namespaces. This also helps in increasing security, for example: a DDoS attack on a Pod in namespace-1 can be isolated to only that namespace without impacting the pods running in other namespaces.

- Limitations:
  - You can't access most resources from another namespace. For e.g. ConfigMap and Secrets are local to a namespace and cannot be shared across different namespaces.
  - Some components of Kubernetes cannot be created within a namespace. For e.g. Persistent volumes and nodes are globally available resources and cannot be added to a namespace.

### Access services within and outside of a namespace

- Default DNS for Kubernetes cluster - svc.cluster.local
- Within a namespace the services can refer to each other just by using service name
  e.g. mysql.connect("db-service")
- To connect with services in the other namespace use the following format:
  e.g. mysql.connect("db-service.dev.svc.cluster.local");
  - db-service -> name of the service in other namespace
  - dev -> name of the namespace where the service is deployed
  - svc -> subdomain for the service
  - cluster.local -> default domain name of Kubernetes cluster
    It is possible to do this because when a new service is created a DNS entry is added to k8s in the above format.

### References

- [Kubernetes documentation - Namespaces](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/)
