---
title: "Kubernetes Fundamentals - Part 4"
date: 2022-10-07
classes: wide
excerpt: "This is part 4 of the 5 part series. Recently, I decided to brush up my Kubernetes skills and the best way to do that is to write a blog post and share my notes with the community. This is a 5 part series where I explain various concepts of Kubernetes at a high level."
header:
  teaser: "https://raw.githubusercontent.com/shivkanwer/shivkanwer.github.io/main/assets/images/kubernetes-fundametals/Kubernetes_architecture.jpg"
categories:
  - Kubernetes
tags:
  - kubernetes
  - containers
---

**This is part 4 of the 5 part series.**  
Recently, I decided to brush up my Kubernetes skills and the best way to do that is to write a blog post and share my notes with the community. This is a 5 part series where I explain various concepts of Kubernetes at a high level.

1. [Part 1](https://thetecktalk.com/kubernetes/kubernetes-fundametals-part1/)
2. [Part 2](https://thetecktalk.com/kubernetes/kubernetes-fundametals-part2/)
3. [Part 3](https://thetecktalk.com/kubernetes/kubernetes-fundametals-part3/)
4. [Part 4](https://thetecktalk.com/kubernetes/kubernetes-fundametals-part4/)
5. [Part 5](https://thetecktalk.com/kubernetes/kubernetes-fundametals-part5/)

**Who is the target audience?**  
The target audience are the people like me who want to brush up their Kubernetes knowledge or people who are newly starting with Kubernetes and want to build a basic understanding of Kubernetes.

**Any pre-requisites?**  
The reader is expected to have basic understanding of Docker, Containerization and Container Registries.

**Does the articles include hands on labs or samples?**  
These articles will focus on the theoretical concepts of Kubernetes, hence there are no hands on labs included with these articles. However, I have provided links to Kubernetes official documentation where you can find good hands on examples about the concepts discussed in these articles.

**Which topics are discussed in part 4?**

1. [Jobs and CronJobs](#jobs-and-cronjobs)
2. [Ingress Controller](#ingress-controller)
3. [Network Policies](#network-policies)
4. [Pod Storage](#pod-storage)
5. [Stateful Sets](#stateful-sets)
6. [Daemon Set](#daemonset)

## **Jobs and CronJobs**

Kubernetes can host a variety of workloads like Web apps, APIs, databases etc. These workloads are meant to continue to run for a long period of time. However, there are other types of workloads like Batch processing, generating reports, performing ETL, sending emails etc. that are meant to carry out a specific task and then finish. When it comes to use cases like Batch processing, generating reports etc. we may need multiple pods to run in parallel to complete the task. At the same time we also want to ensure that all pods perform the task assigned to them successfully and then exit.

Kubernetes Job is like a Manager which can create as many pods as we want to get the work done and ensure that the work gets done successfully.

Why not use ReplicaSet to run multiple pods?  
While a Replica Set is used to make sure a specified number of pods are running at all times, a Job is used to run a set of pods to perform a given task to completion.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 5 # Run the job until 5 successful completions
  parallelism: 2 # Run two pods in parallel
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ["expr", "3", "+", "4"]
      restartPolicy: Never # Important, otherwise Kubernetes will restart the pod even if it exits due to successful task completion
  backoffLimit: 4 # Specify the number of retries before considering a Job as failed. Default is 6
```

A `restartPolicy` of `Never` or `OnFailure` is allowed for a Job.
{: .notice--info}

A job that can be scheduled is called _CronJob_. CronJobs are meant for performing scheduled actions such as taking backups of the database once every week, generating a report every morning at 9 AM, and so on. Each of those tasks should be configured to recur indefinitely, you can define the point in time within that interval when the job should start.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *" # Cron Schedule Syntax
  jobTemplate:
    spec: # spec section of the Job
      template:
        # Complete spec section of Pod (including the spec property)
```

Cron schedule syntax

```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │                                   OR sun, mon, tue, wed, thu, fri, sat
# │ │ │ │ │
# * * * * *
```

## Ingress Controller

Kubernetes allows you to expose your services directly to the internet using the service of type Load balancer or NodePort. It will directly assign a Public IP to your service, and you can access it using that IP.
However, the Kubernetes load balancer service is a Layer 4 load balancer. Layer 4 load balancers only deal with routing decisions between IP addresses, TCP, and UDP ports. The Service is unaware of the actual applications and cannot make any other routing considerations. And chances are that in a real production scenario you may need a LB that has features like reverse proxy, configurable traffic routing, SSL termination, URL, or path-based routing etc.

This is where the ingress controller comes into picture. It is software that provides layer 7 load balancer features like reverse proxy, configurable traffic routing, and TLS termination for Kubernetes services. Another advantage is that the ingress configuration can be defined using the Kubernetes manifest files and hence can be easily managed along with rest of the application.

Ingress controllers work at layer seven and can use more intelligent rules to distribute application traffic. Everyday use of an Ingress controller is to route HTTP traffic to different applications based on the inbound URL.

<p align="left">
  <img src="/assets/images/kubernetes-fundametals/ingress.svg" style="max-width: 750px;">
</p>

Kubernetes cluster does not come with an ingress controller by default, so you have to deploy one. There are several options for running ingress on Kubernetes such as Istio, HAProxy, Kong, NGINX, Traefik etc.

Ingress resource - The routing rules that we create for routing the traffic to backend services.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
spec:
  ingressClassName: nginx #defines that we are using NGINX ingress controller
  rules:
    - host: "theteckie.com" # This is an optional parameter, If not specified then the default value of * will be considered
      http: # This attribute represents the internal service protocol (HTTP/HTTPs). This protocol will be used for forwarding the request to the internal service. Do not confuse this with the URL + Protocol the users will use in the browser
        paths:
          - path: /testpath
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80
```

## Network Policies

By default the network in Kubernetes is flat which means that any pod can accept communication from any other pod in the cluster (even if the pods are in different namespaces). When you run modern, microservices-based applications in Kubernetes, you often want to control which components can communicate with each other. The principle of least privilege should be applied to how traffic can flow between pods in an Kubernetes cluster. The Network Policy feature in Kubernetes lets you define rules for ingress and egress traffic between pods in a cluster.

For example, say you have deployed the front-end, API and database pods on the Kubernetes cluster. You want that the database pod should only accept traffic from API pod and not directly from the front-end Pod.

<p align="left">
  <img src="/assets/images/kubernetes-fundametals/networkpolicy.svg" style="max-width: 400px;">
</p>

This is where you can use the _Network Policy_ and create a rule on the database pod which defines that it should only accept traffic from the API Pod. But what about the response from the database? Do you need to a separate rule for that? No, once you allow the incoming traffic on a certain port, the response to the traffic is allowed back automatically. So, you don't need to create a separate egress rule for that.

However, this does not mean that the database pod can now make an outbound call to the API pod. For any traffic originating from the database pod, you will need to create an explicit egress rule to allow that traffic.

A single Network policy can be used for defining rules for both ingress as well as egress rules. To implement a network policy you will need to install a Network Plugin on the Kubernetes cluster. There are several network plugins available in the market like Calico, Weave-net etc.

This is how the network policy looks like:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector: # Select the Pod on which to apply the Network Policy. If let empty, this network policy applies to all pods in the namespace
    matchLabels:
      role: db
  policyTypes: # If no policyTypes are specified on a NetworkPolicy then by default Ingress will always be set and Egress will be set if the NetworkPolicy has any egress rules.
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock: # Rule 1
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
          # OR
        - namespaceSelector: # Rule 2
            matchLabels:
              project: myproject
          # OR
        - podSelector: # Rule 3
            matchLabels:
              role: frontend
          # AND
          namespaceSelector:
            matchLabels:
              name: prod
      ports:
        - protocol: TCP
          port: 6379
      # The FROM rules are evaluated as follows, in this case the Pod can receive traffic on port 6379 from:
      # - IP addresses in the ranges 172.17.0.0–172.17.0.255 and 172.17.2.0–172.17.255.255 (ie, all of 172.17.0.0/16 except 172.17.1.0/24)
      # - any pod in a namespace with the label "project=myproject"
      # - any pod in the "prod" namespace with the label "role=frontend"
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978
    # The TO rule here allows connections from any pod in the "default" namespace with the label "role=db" to CIDR 10.0.0.0/24 on TCP port 5978
```

When you define only the ingress policy, then by default there is no impact on egress traffic. If you need isolation for egress traffic then you need explicitly define the `Egress` rules in the network policy.

## Pod Storage

Containers are meant to be transient in nature, which means that they can come and go at any time. Once a container is terminated, any data saved inside the container gets lost.  
To persist data generated by the containers we attach the Volumes to the containers. Volumes represent an external storage which is not dependent on the lifecycle of a container. Even if the container is destroyed the data is safe in the Volume.

### Volume Types

Kubernetes supports several different storage solutions which can be mounted as volumes on the containers like AWS Elastic Blob Store, Azure File Storage, Azure Disk Storage, NFS, GlusterFS, HostPath, Flocker etc.

### Persistent Volumes (PV)

Managing storage is a distinct problem from managing compute instances. Usually we would like to manage the storage centrally where the Admin can create a large pool of storage and then have users carve out pieces from it as required. This is where Persistent volume can help us. A persistent volume is a cluster wide pool of storage configured by an Admin to be used by users deploying applications on the cluster. PVs have a lifecycle independent of any individual Pod that uses the PV. A PersistentVolume can be statically created by a cluster administrator, or dynamically created by the Kubernetes API server.

Create a persistent volume statically:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  accessMode:
    - ReadWriteOnce # Supported Values - ReadWriteOnce, ReadOnlyMany, ReadWriteMany
  capacity:
    storage: 10Gi
  persistentVolumeReclaimPolicy: Retain # The reclaim policy for a PersistentVolume tells the cluster what to do with the volume after it has been released of its claim
  # Supported Values:
  # Retain (Default) - Admin will do manually cleanup
  # Delete - Deletion removes both the PersistentVolume object from Kubernetes, as well as the associated storage asset in the external infrastructure
  # Recycle - This performs a basic scrub on the volume and makes it available again for a new claim.
  storageClassName: managed-csi # This will change based on the Volume Type that you are using for your Kubernetes cluster like Azure Files, AWS Elastic Blob Store etc.


  # Other properties in the YAML file will vary based on the storage solution that you are using for your Kubernetes cluster.
```

PVs can be provisioned in two ways:

- Static : A cluster administrator creates a number of PVs (using YAML definition files as explained above). They carry the details of the real storage, which is available for use by cluster users.
- Dynamic: The PV is automatically created when a new PVC gets created. This is done through the use of Storage Classes. Different Storage providers like AWS, GCP, Azure etc. have different storage classes defined for their respective storage solutions. When creating a PVC we can directly use the Storage Class to define which storage solution to be used and it will automatically create a PV and bind it to the PVC.

### Persistent Volume Claims (PVC)

A PersistentVolumeClaim (PVC) is a request for storage by a user. It is similar to a Pod, a Pod consume node resources and PVC consume PV resources. An Admin creates a set of PVs and a user creates PVCs to use the storage. Once the PVC is created Kubernetes binds the PV to PVC based on the request. During the binding process Kubernetes tries to find a PV that has sufficient capacity as requested by the claim. Kubernetes also takes into consideration other factors like accessMode, storageClass, Selector etc.

There is a 1:1 relationship between PV and PVC, if the PVC does not consume the full capacity of the PV then that left over capacity cannot be used by other PVCs.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim
spec:
  # Specify the criteria for PVC like accessMode, required capacity etc.
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

Mount the PVC on to the container inside a Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - mountPath: "/objects/data"
          name: vol1
  volumes:
    - name: vol1
      persistentVolumeClaim:
        claimName: pv-claim
```

### Storage Classes

With Storage Classes you can define a provisioner such as Google Storage, Azure Disk Storage, Azure File Storage etc. that can automatically provision the required storage on GCP or Azure (or other storage providers) and attach that to pods when a claim is made. This is called dynamic provisioning of volumes.

Each StorageClass contains the fields provisioner, parameters, and reclaimPolicy, which are used when a PersistentVolume belonging to the class needs to be dynamically provisioned.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-storage-class
provisioner: files.csi.azure.com # This value depends upon the choice of storage provider
parameters: # Parameters depend upon the choice of storage provider
  skuName: Standard_LRS
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
# Supported values - Immediate (Default) or WaitForFirstConsumer (delay the binding and provisioning of a PersistentVolume until a Pod using the PersistentVolumeClaim is created)
allowVolumeExpansion: true # Allows the volume to be resized
```

To use the storage class in the PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: my-storage-class
```

So, next time when a PVC is created it knows which provisioner to use for provisioning the required amount of storage. The PVC first uses the provisioner to create the required storage (like an Azure disk), then creates a persistent volume and then binds the PVC to the persistent volume. So, the PV still gets created but you don't have to create it manually anymore.

## Stateful sets

Stateful sets are very similar to deployments in the sense that you can deploy pods using templates, create replicas, perform rolling updates and rollbacks etc. But there are some differences:

- In Stateful sets the pods are deployed sequentially, after the first pod is deployed it must be in a running state before the next pod is deployed. Whereas in deployments all the pods are deployed at once and their is no way to guarantee a certain order of deployment for the pods.
- During deployment the pods are assigned random names, if the pod crashes and a new pod gets created in its place it is given another random name, so we cannot rely on pod name. On the other hand stateful sets assign a unique ordinal index to each pod starting with 0. A pod is assigned a static name which is a combination of name of stateful set and index, for example mysql-0, mysql-1 and so on. We can rely on the pod names. Even if the pod fails and is recreated it would still be assigned the same name. Stateful sets maintain a sticky identity for each of its pods.
- StatefulSets currently require a Headless Service to be responsible for the network identity of the Pods.

## DaemonSet

A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.

Some typical uses of a DaemonSet are:

- running a cluster storage daemon on every node
- running a logs collection daemon on every node
- running a node monitoring daemon on every node

### Reference

- [Kubernetes documentation - Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Kubernetes documentation - Network Policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
