---
title: "Kubernetes Fundamentals - Part 1"
date: 2022-10-06
classes: wide
categories:
  - Kubernetes
tags:
  - kubernetes
  - containers
---

Recently, I decided to brush up my Kubernetes skills and the best way to do that is to write a blog post and share my notes with the community. This is a 4 part series where I explain various concepts of Kubernetes at a high level.

1. [Part 1](https://theteckie.com/kubernetes/kubernetes-fundametals-part1/)
2. [Part 2](https://theteckie.com/kubernetes/kubernetes-fundametals-part2/)
3. [Part 3](https://theteckie.com/kubernetes/kubernetes-fundametals-part3/)
4. Part 4 (Coming soon!)

**Who is the target audience?**  
The target audience are the people like me who want to brush up their Kubernetes knowledge or people who are newly starting with Kubernetes and want to build a basic understanding of Kubernetes.

**Any pre-requisites?**  
The reader is expected to have basic understanding of Docker, Containerization and Container Registries.

**Does the articles include hands on labs or samples?**  
These articles will focus on the theoretical concepts of Kubernetes, hence there are no hands on labs included with these articles. However, I have provided links to Kubernetes official documentation where you can find good hands on examples about the concepts discussed in these articles.

**Which topics are discussed in part 1?**

1. [Kubernetes Architecture](#kubernetes-architecture)
2. [Pods](#pods)
3. [ReplicaSet](#replicaset)
4. [Deployment](#deployments)
5. [Services](#services)
6. [Namespace](#namespace)

## **Kubernetes Architecture**

<p align="center">
  <img alt="Kubernetes Architecture Diagram" src="/assets/images/kubernetes-fundametals/Kubernetes_architecture.svg">
</p>

A Kubernetes cluster consists of different types of nodes. These nodes are nothing but the usual physical or virtual machines. These nodes are split into two types - the **Master nodes**, which hosts the Kubernetes platform that controls and manages the whole Kubernetes system. And then there are the **Worker nodes** that run the actual containers which we deploy. The master node, is what controls the cluster and makes it function. It is what we call the heart and soul of Kubernetes.

The Master node also known as Control Plane, is responsible for making global decisions about the cluster (for example, scheduling the pods), as well as detecting and responding to various cluster level events. The Control Plane has four important components:

1. _API Server_ is the front-end of the Kubernetes Control Plane. This is what you and the other control plane components communicate with. The API server is how the underlying Kubernetes APIs are exposed. This component provides the interaction for management tools, such as kubectl or the Kubernetes dashboard.
2. _Scheduler_ is responsible for assigning a worker node to each deployable component of your application. It keeps track of how much resources are available on which worker node and then schedules the Pods accordingly.
3. _Controller Manager_ performs the cluster-level functions, such as replicating component, deployments, keeping a track of worker nodes, handling node failures, and so on. So, that is why we generally refer to controller as the brains of Kubernetes.
4. _etcd_ is a reliable distributed data store that persistently stores the cluster configuration. It is a database for K8s where it stores all cluster related information – Job scheduling info, Pod details etc.

The components of the master node hold and control the state of the cluster, but usually we don't run applications on Master nodes. For this we use the worker nodes. Each worker node consists of three important components:

1. _Kubelet_ handles the communication between master nodes and worker nodes. It is an agent that interacts with the API server in the Master Node and receives the deployment information for spinning up new pods.
2. _Container Runtime_ is responsible for running the containers. Kubernetes is compatible with many different container runtimes like Docker, rkt, Moby etc.
3. _Kube-proxy_ is the network proxy that runs on each node and maintains the network rules. These network rules allow network communication to your Pods from other network inside or outside of your cluster.

## **Pods**

Pod is the smallest object that can be created in Kubernetes. A pod is a single instance of an application. Containers are encapsulated within a Pod and single Pod can have multiple containers. Each container within the pod share the underlying resources like network, storage, state, CPU, Memory etc.

- Containers within the same pod can reach each other using localhost.
- Scaling in Kubernetes is done at the Pod level and not at the container level.
- Kubernetes always restarts the pod after it exits and this happens due to the `restartPolicy` property. By default the restart policy is set to `Always` which means even if the pod exits due to completion of its task, k8s still attempts to restart that pod. To overcome this you can set the `restartPolicy` to `Never` or `OnFailure`.

## **ReplicaSet**

ReplicaSet ensure that a desired number of instances of a pod are always running (even if the desired number is 1). This allows us to ensure high availability for our application. Another reason we need ReplicaSet is to create multiple pods to share the user load. As the demand grows we can increase the replica count and scale our application.

ReplicaSet can also be created to monitor existing pods, but you still need to provide the YAML definition of existing pods.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
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

Deployment is the preferred way of deploying applications to Kubernetes. Deployments offer features like rolling updates, rollout history, rollback to previous version etc. With deployment you can make multiple changes to the environment in one go like scaling the number of replicas, changing container image version, change container resource allocation etc. and the deployment will automatically take care of rolling out these changes.

From YAML definition perspective the only diff. between `Deployment` and `ReplicaSet` is the _kind_ parameter which needs to be set to "Deployment" instead of "ReplicaSet". Rest of the properties are same as ReplicaSet definition.

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

When you create a new deployment it creates a _Rollout_ which creates a _Revision_. Every time you update your application through a deployment a new _Revision_ gets created. This helps Kubernetes keep track of the changes made to a deployment and enables you to perform rollback to any of the previous version if required.

```sh
# View deployment status
kubectl rollout status deployment my-dep

# View deployment history
kubectl rollout history deployment my-dep

# Rollback or undo a deployment
kubectl rollout undo deployment my-dep
```

### Deployment Strategies

There are two types of deployment strategies supported by Kubernetes:

1. _Recreate Strategy_ - Destroy all existing instances of the application at once and bring up the new version of the application [This will result in downtime]
2. _Rolling update_ (default strategy) - The existing pods are brought down one by one and newer version is deployed at the same time. [No application downtime]

How does Rolling update strategy works?  
When you deploy a newer version of your application, Kubernetes creates a new `ReplicaSet` and starts deploying the new pods in that ReplicaSet. At the same time as new Pods are getting deployed one by one, Kubernetes also brings down the older pods in the old ReplicaSet. Once the old ReplicaSet is empty, k8s deletes that ReplicaSet. The Rollback features also works in the same way but in reverse.  
Below is a screenshot describing how rolling update brings down each Replica one at a time and deploys the new replica. Here the replica count is 5:

<img src="/assets/images/kubernetes-fundametals/RollingUpdate.png" style="max-width: 1000px;">

<br/>
Below screenshot shows how the Recreate strategy brings down all the old replica at once and then deploys the new replica:
<img src="/assets/images/kubernetes-fundametals/Recreate.png" style="max-width: 900px;">

### Max Unavailable and Max Surge in a Rolling update strategy

- _Max Unavailable_ - Deployment ensures that only a certain number of Pods are down while they are being updated. By default, it ensures that at least 75% of the desired number of Pods are up (25% max unavailable).
- _Max Surge_ - Deployment also ensures that only a certain number of Pods are created above the desired number of Pods. By default, it ensures that at most 125% of the desired number of Pods are up (25% max surge).

For example, if you observe a Deployment closely, you will see that it first creates a new Pod, then deletes an old Pod, and creates another new one. It does not kill old Pods until a sufficient number of new Pods have come up, and does not create new Pods until a sufficient number of old Pods have been killed. For a deployment with 3 replicas, it makes sure that at least 2 Pods are available and that at max 4 Pods in total are available. In case of a Deployment with 4 replicas, the number of Pods would be between 3 and 5.

### Blue/Green and Canary Deployment Strategy

Apart from the Recreate and Rolling update deployment strategies you can also follow other deployment strategies like Blue/Green deployment, Canary deployment for your applications. These strategies are not something that you can configure in the deployment definition file but represent a different way of releasing your application. These deployment strategies are best implemented with service meshes like Istio.

_Blue/Green Deployment_: In this strategy a new version of the application is deployed along with the old version. The old version is call _Blue_ and new version is called _Green_. Once the new version is deployed, 100% of the traffic is still routed to the Blue version while the tests are being run on the Green version. Once all tests are passed, traffic is switched to Green version all at once.

_Canary Deployment_: In this strategy we deploy a new version of the application along with the old version and route a small percentage of traffic to the new version. Once all the tests are successful, we upgrade the older version with the newer version of the application (which can be done by Rolling update strategy, for example). Finally, we remove the canary deployment which was deployed for running the tests.

## **Services**

Kubernetes assigns an IP address to every Pod and the containers inside the Pod share that IP address. However, pods are regularly created and destroyed, causing their IP addresses to change constantly. This results in discoverability issues since it becomes difficult for the applications to connect with a pod if its IP address keeps on changing. So how do we manage communication with pods? How do we manage traffic routing or load balancing?  
This is where the concept of Kubernetes _Service_ comes into picture. Service enable communication between various components both inside and outside of the Kubernetes Cluster. Service keep track of the changes in IP addresses and DNS names of the pods and expose them through a single IP or DNS. It provides:

- Stable IP address
- Traffic routing to the Pods
- Load balancing
- Loose coupling
- Internal or external communication

A Service is not a Pod, it is an abstraction layer that represents an IP address.

There are three types of services in Kubernetes:

1. _NodePort_ services makes a pod accessible over a particular port on the node. There are three ports involved when creating a NodePort service. These ports need to be looked from the view point of a service:

   1. _TargetPort_: This is the Port at which the Pod is exposing the application. If we don't provide a target port, it is assumed to be same as the "Port".
   2. _Port_: This is the port to be opened on the service itself.
   3. _NodePort_: This is the Port which will be opened on the node. If we don't provide the NodePort then a free port in the valid range between 30000-32767 is allocated as NodePort.

    <p align="center">
      <img alt="Kubernetes node port service" src="/assets/images/kubernetes-fundametals/nodeport.svg" style="max-height: 600px;">
    </p>

   NodePort services can automatically distribute traffic across multiple Pods, it acts as a LoadBalancer. It also works even if the pods are distributed across multiple nodes in the cluster. Kubernetes automatically creates a NodePort service spanning all the nodes in the cluster. You can use IP address and port on any node to connect to the Pod:

    <p align="center">
      <img alt="Kubernetes node port service expanding multiple nodes" src="/assets/images/kubernetes-fundametals/nodeport_multinode.svg" style="max-height: 600px;">
    </p>

2. _ClusterIP_ service type exposes the services on an internal IP address. It makes the Service only reachable from within the cluster. This is the default service type. It is also known as _Internal Service_.
3. _LoadBalancer_ service type creates a Load balancer in the supported Cloud Provider like Azure, AWS, GCP etc.

## **Namespace**

Namespace is like a virtual sub cluster inside a Kubernetes cluster. You can have multiple namespaces in a Kubernetes cluster and they are isolated from each other. Namespaces can help you with Organization, Security and Performance of your containerized workloads. You can also specify resource limitations at the namespace level.

When you create a new Kubernetes cluster, following namespaces are created by default:

- _default_ - This is an empty namespace which can be used by you for deploying your own applications.
- _kube-system_ - This namespace contains the components that are used for managing the system processes and you should not make any changes to this namespace.
- _kube-public_ - This namespace contains publicly accessible data. It contains a config map which contains cluster information which is accessible without authentication. This is the information which you get when you type 'kubectl cluster-info'.
- _kube-node-lease_ - This is a recent addition to Kubernetes. This namespace holds information about the heartbeats of the nodes. Each nodes has an associated lease object in this namespace that contains information about that node's availability.

Use cases for namespaces:

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

### References

- [Ref. link with good comparison between different types of services](https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0)
- [Kubernetes documentation - Service](#https://kubernetes.io/docs/concepts/services-networking/service/)
- [Kubernetes documentation - Namespaces](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/)
