---
title: "Kubernetes Fundamentals - Part 3"
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

**Which topics are discussed in part 3?**

1. [Taints and Tolerations](#taints-and-tolerations)
2. [Assigning Pods to Nodes](#assigning-pods-to-nodes)
3. [Multi-container Pod Patterns](#multi-container-pod-patterns)
4. [Readiness, Liveness and Startup Probes](#readiness-liveness-and-startup-probes)
5. [Monitoring](#monitoring)
6. [Labels, Selectors and Annotations](#labels-selectors-and-annotations)

## **Taints and Tolerations**

`Taints` and `Tolerations` define the relationship between pods and Nodes. It is a way to control what nodes can accept what pods. Taints are a kind of constraints applied on the node, if a Pod is intolerant to that taint then it cannot be scheduled on that node.

Taints and tolerations have nothing to do with security or intrusion in Kubernetes cluster.
{: .notice--info}

_Taints_ are applied on the nodes in Kubernetes cluster. A node can have multiple taints.

For example, To taint a node to only accept Pods for frontend application, add a taint to node as follows:

```yaml
kubectl taint node node1 app=front:NoSchedule
```

Taint effect defines what happens to the Pods that do not tolerate a particular taint. There are three types of taint effects:

1. NoSchedule - The scheduler does not place any intolerant pods on the node.
2. PreferNoSchedule - The scheduler will try to avoid placing an intolerant pod on the node, but that is not guaranteed.
3. NoExecute - New Pods that are intolerant to the taints on the nodes will not be scheduled on that node and any existing pods running on that node will be evicted in case they are intolerant to the taints.

_Tolerations_ are applied at the Pod level. A Pod can define multiple tolerations.

For example, To allow a Pod to be scheduled on the node which has taint of `app=frontend:NoSchedule`, define the following toleration in the Pod definition file:

```yml
tolerations:
  - key: "app"
    value: "frontend"
    effect: "NoSchedule"
    operator: "Equal"

# OR you can also use Exists operator instead of Equal to check if a certain taint exists on the node

tolerations:
  - key: "app"
    effect: "NoSchedule"
    operator: "Exists"

# In the above example no value needs to be specified since we are only checking if a certain taint exists on the node or not
```

Taints and tolerations are only meant to restrict nodes from accepting certain pods, but it does not guarantee that a pod with specific tolerations will always be placed on a node with matching taints. To deploy a pod on specific node use the `Node Selector` or `Node Affinity` attributes (discussed in next section)
{: .notice--info}

Why the scheduler does not schedule any pods on the master node?  
This is because when a Kubernetes cluster is setup the master nodes are configured with a taint `node-role.kubernetes.io/master:NoSchedule`. This taint prevents the scheduler from deploying any pods to the master nodes. However, this behavior can be modified.

## **Assigning Pods to Nodes**

In the previous sections we saw how a node can be configured to accept only certain pods through the use of Taints and Tolerations. But, how do we ensure that a certain Pod should run on a specific node?
In Kubernetes you can use any of the following methods to achieve this:

1. Node Selector
2. Node Affinity
3. Node Name

### Node Selector

Node Selector works on the concept of labels and selector. A node can be labelled with particular key/value pair and then a Pod can use that key/value pair to specify which node it should get scheduled on. Kubernetes also automatically adds certain labels like: `kubernetes.io/os=linux` to each node when it gets created and these labels can be used as `nodeSelector` in the Pod definition.

```yml
# To schedule a Pod on windows node
spec:
  nodeSelector:
    kubernetes.io/os: windows # This label is already added by Kubernetes to Windows worker nodes
  containers:
    - name: my-webapp
      image: my-webapp:v1
```

You can add custom labels to the node as well:

```yaml
# Add label to a node
kubectl label node node-01 app=frontend

# Use the above label to schedule a Pod on this node
spec:
  nodeSelector:
    app: frontend

# Multiple labels can be specified, but all labels should match the labels on the node
spec:
  nodeSelector:
    app: frontend
    size: large
```

One of the limitations of Node Selector is that you cannot specify complex criteria to schedule your pods. Node Selector only selects nodes with all the specified labels. Imagine if you have a complex scenario like schedule the pod on a node with size large or medium but not small. Or something like, schedule the pod on a node with higher memory capacity or an SSD. These types of expressions cannot be specified through node selector. This is where _Node Affinity_ comes into picture.

### Node Affinity

Node Affinity feature provides us advanced capabilities to limit pod placement onto specific nodes. Node affinity functions like the nodeSelector field but is more expressive and allows you to specify rules.
There are three types of node affinity:

1. `requiredDuringSchedulingIgnoredDuringExecution` - The scheduler can't schedule the Pod unless the rule is met. IgnoredDuringExecution means that if the node labels change after Kubernetes schedules the Pod, the Pod continues to run.
2. `preferredDuringSchedulingIgnoredDuringExecution` - The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod. IgnoredDuringExecution means that if the node labels change after Kubernetes schedules the Pod, the Pod continues to run.
3. `requiredDuringSchedulingRequiredDuringExecution` [Planned but not yet available] - The scheduler can't schedule the Pod unless the rule is met. RequiredDuringExecution means that if the node labels change after Kubernetes schedules the Pod, the Pod will be evicted from the node unless the node affinity is updated for that Pod.

For example, let's say the nodes in the cluster are label as large, medium and small based on their resources.

```yml
# Schedule the Pod on either large or medium node
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: size
              operator: In # You can use In, NotIn, Exists, DoesNotExist, Gt and Lt
              values:
                - large
                - medium

# OR
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: size
              operator: NotIn
              values:
                - small
```

### Node Name

This is the simplest way to express on exactly which node you want to run your pod on, you just directly specify the name of the node, as shown in below example:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
  nodeName: node-01
```

However, this approach has some limitations:

- If the named node does not exist, the Pod will not run, and in some cases may be automatically deleted.
- If the named node does not have the resources to accommodate the Pod, the Pod will fail and its reason will indicate why, for example OutOfmemory or OutOfcpu.
- Node names in cloud environments are not always predictable or stable.

## **Multi-container Pod Patterns**

There are three common multi-container patterns in Kubernetes:

1. Sidecar pattern: Sidecar containers extend and enhance the "main" container, they take existing containers and make them better. For example, logging utilities, sync services, watchers, monitoring agents.
2. Adaptor pattern: Adapter containers standardize and normalize output. Consider the task of monitoring n different applications. Each application may be built with a different way of exporting monitoring data but every monitoring system expects a consistent and uniform data model for the monitoring data it collects. By using the adapter pattern, you can transform the heterogeneous monitoring data from different systems into a single unified representation.
3. Ambassador pattern: Ambassador container is a useful way to connect containers with the outside world. An ambassador container is essentially a proxy that allows the other container to connect to a port on localhost while the ambassador container can proxy these connections to different environments depending on the cluster's needs.

### Init Containers

Init containers are specialized containers that run before the actual application container. These containers usually contain the setup scripts or utilities which are required to be run before running the actual application.

For example, a container that pulls code or binary from a repository that will be used by the main web application. This container is only required to be run when the Pod is first created. Or a container that waits for an external service or database to be up before the actual application starts.

Init containers can also enhance the security by keeping unnecessary tools separate from the main application and this helps you to limit the attack surface of your app container image.

When a POD is first created the init container is run, and the process in the init container must run to a completion before the real container hosting the application starts. You can configure multiple such init containers as well. In that case each init container is run one at a time in sequential order.

If any of the init containers fail to complete, Kubernetes restarts the Pod repeatedly until the init container succeeds.

An init container is configured in a pod like all other containers, except that it is specified inside a `initContainers` section, like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: myapp
spec:
  containers:
    - name: my-container
      image: busybox:1.28
      command: ["sh", "-c", "echo The app is running! && sleep 3600"]
  initContainers:
    - name: warm01 # This container is run before the actual application container
      image: busybox
      command:
        [
          "sh",
          "-c",
          "git clone <some-repository-that-will-be-used-by-application> ;",
        ]
    - name: warmup02
      image: busybox
      command: ["sleep", "20"]
```

## **Readiness, Liveness and Startup Probes**

Probes are used for checking the status of the Pods. There are three types of probes in Kubernetes:

1. Readiness Probe
2. Liveness Probe
3. Startup Probe

### Readiness Probe

The Readiness probes indicates whether the Pod is ready to respond to requests or not. If the readiness probe fails the endpoint controller removes the Pod's IP address from the service so that no traffic is routed to the Pod until it is ready. If you do not specify readiness probe then by default the Pod is put into ready state as soon as all its containers are up and running.

An important point to note is readiness probe is meant to check if the application running inside the container is actually ready to receive the traffic or not. A container in a running state does not guarantee that the application inside the container is also up and running. An application may require more time to start (like spin up the web server, populate cache, establish database connectivity, load large data or configuration files during startup etc.). Hence, it is the responsibility of the application developer to implement some mechanism in the application to indicate if it is ready to receive traffic or not. This process/mechanism can then be probed by Kubernetes to check the state of the application and take appropriate action.

```yml
# Create an HTTP Readiness Probe
spec:
  containers:
    - name: my-web-app
      image: my-web-app:v1
      readinessProbe:
        httpGet:
          path: /api/isReady # This API needs to be implemented by the developer
          port: 80
        initialDelaySeconds: 10 # Wait for 10 seconds before checking the readiness endpoint
        periodSeconds: 3 # This parameter indicates how often should Kubernetes query the readiness endpoint
        failureThreshold: 5 # After this threshold Kubernetes will stop querying the readiness endpoint

# Create a TCP Readiness Probe
spec:
  containers:
    - name: my-web-app
      image: my-web-app:v1
      readinessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 3
        failureThreshold: 5

# Create a Readiness probe that checks readiness by executing a command in the container
spec:
  containers:
    - name: my-web-app
      image: my-web-app:v1
      readinessProbe:
        exec:
          command:
            - cat
            - /tmp/healthy
        initialDelaySeconds: 10
        periodSeconds: 3
        failureThreshold: 5
```

Readiness probes runs on the container during its whole lifecycle. If the readiness probe fails, Pod can no longer server the traffic. However, in this case Kubernetes will not automatically restart the container inside the Pod.

### Liveness Probe

Indicates whether the container is running or not. Again, by container we mean the actual application inside the container is running or not. If the liveness probe fails, the kubelet kills the container, and the container is subjected to its restart policy (`Always`, `Never`, `OnFailure` - The default value is Always).
Here again the application developer is responsible for implementing the appropriate health checks which can be used by Kubernetes to determine if the application is alive or not.
Both readiness and liveness probes can be specified for the same container.

```yml
spec:
  containers:
    - name: my-web-app
      image: my-web-app:v1
      readinessProbe:
        httpGet:
          path: /api/isReady # This API needs to be implemented by the developer
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 3
        failureThreshold: 5
      livenessProbe:
        httpGet:
          path: /api/isAlive # This API needs to be implemented by the developer
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 3
        failureThreshold: 5
# The liveness probe can also be used for querying tcp endpoint or executing a query in the container just like readiness probe. The syntax is also same.
```

### Startup Probe

In scenarios where you are dealing with applications that require long time to startup, like a legacy application loading a lot of libraries or data during application startup. In these cases it might be bit tricky to setup the liveness or readiness probes. The startup probe indicates whether the application within the container is started. All other probes are disabled if a startup probe is provided, until it succeeds. If the startup probe fails, the kubelet kills the container, and the container is subjected to its restart policy.
The startup probe provides a longer time interval for the application to start which is equivalent to `failureThreshold * periodSeconds`.

```yml
startupProbe:
  httpGet:
    path: /api/isStarted
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
# In this case the application will have a maximum of 5 minutes (30 * 10 = 300s) to finish its startup. Once the startup probe has succeeded, the liveness probe takes over. If the startup probe never succeeds, the container is killed after 300s and subject to the pod's restartPolicy.
```

## **Monitoring**

As of today, Kubernetes does not come with a full featured built-in monitoring solution. However, there are many open source and propriety solutions available like - Metrics Server, Prometheus, DataDog, Elastic Stack etc.

### Metrics Server

Metrics Server is an in-memory monitoring solution, it does not store the metrics on the disk. As a result you cannot do historical data analysis, for that you need to rely on one of the other monitoring solutions. You can enable only 1 Metrics server per Kubernetes cluster.

How does the Metrics Server works?  
Each node of Kubernetes cluster runs an agent known as _kubelet_ which is responsible for communication with the master nodes and running pods on the nodes. The _kubelet_ contains a component known as _cAdvisor (or Container Advisor)_ which is responsible for collecting performance metrics from the pods and node and sending it to the Metrics Server through the kubelet API.

```yaml
# To view performance metrics of nodes
kubectl top node

# To view performance metrics of pods
kubectl top pod
```

### Container Logs

To view containers logs in Kubernetes use following commands:

```yaml
# To view the container logs
kubectl logs pod-name

# To stream the container logs live
kubectl logs -f pod-name

# If a Pod has more than one containers, then specify the container name in the command to view the logs from a specific container
kubectl logs -f pod-name container-name
```

## **Labels, Selectors and Annotations**

Labels and Selectors are a standard method to group things together in Kubernetes.  
_Labels_ are properties attached with each Kubernetes Object. You can add as many labels to an object as you want.

```yaml
# For example, A frontend Pod
labels:
  tier: front-end
  app: web
```

When discovering a kubernetes object using _Selector_, you can specify one or more labels to help discover that object.

```yaml
# For example, Select the Pod where tier is front-end and app is web.
selector:
  matchLabels:
    tier: front-end
    app: web
```

```bash
# Select Pods with particular labels:
kubectl get pod --selector env=dev # env=dev is the label on the pod

# Get a pod with multiple labels
kubectl get pod --selector env=prod,tier=frontend
```

_Annotations_ are used for recording details for informatory purposes.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  Labels:
    app: finance
    tier: front-end
  annotations:
    buildVersion: 1.34
    releaseDate: 09-02-2022
    owner: testowner@abc.com
spec:
  replicas:
  template:
  selector:
```

### References

- [Kubernetes documentation - Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
- [Kubernetes documentation - Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)
- [Kubernetes documentation - Assigning Pods to Nodes using Node Affinity](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/)
