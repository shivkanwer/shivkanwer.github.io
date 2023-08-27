---
title: "Kubernetes Fundamentals - Part 5"
date: 2022-10-07
classes: wide
excerpt: "This is part 5 of the 5 part series. Recently, I decided to brush up my Kubernetes skills and the best way to do that is to write a blog post and share my notes with the community. This is a 5 part series where I explain various concepts of Kubernetes at a high level."
header:
  teaser: "https://raw.githubusercontent.com/shivkanwer/shivkanwer.github.io/main/assets/images/kubernetes-fundametals/Kubernetes_architecture.jpg"
categories:
  - Kubernetes
tags:
  - kubernetes
  - containers
---

**This is part 5 of the 5 part series.**  
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

**Which topics are discussed in part 5?**

1. [Kubernetes REST APIs](#kubernetes-rest-apis)
2. [Authentication, Authorization and Admissions Control](#authentication-authorization-and-admissions-control)
3. [Custom Resource Definition (CRD), Custom Controllers and Operators](#custom-resource-definition-crd-custom-controllers-and-operators)
4. [Helm](#helm)

## Kubernetes REST APIs

### API Groups

REST API is the fundamental fabric of Kubernetes. All operations and communications between components, and external user commands are REST API calls that the API Server handles.  
The APIs are divided into several API groups such as:

```bash
/metrics
/healthz
/version
/api
/apis
/logs
```

The APIs responsible for cluster functionality are divided into two API groups - `/api` and `/apis`

- The core (also called legacy) group is found at REST path `/api/v1`. This is where the core functionality of kubernetes exists such as Pods, namespaces, services, replicas, nodes, secrets, PVC, PV etc.
- The named groups are at REST path `/apis/$GROUP_NAME/$VERSION`. These are more organized and going forward all the newer features will be made available through named groups. There are several groups under `/apis` like:

  ```bash
  /apps - For e.g. /apps/v1 contains Deployments, ReplicaSets, StatefulSets
  /extensions
  /networking.k8s.io
  /storage.k8s.io
  /authentication.k8s.io
  /certificate.k8s.io
  ```

To access the remote Kubernetes API server from local machine:

```bash
kubectl proxy 8001 # This command launches a local proxy for API server at 127.0.0.1:8001 and uses the credentials from the kubeconfig file

curl http://localhost:8001/version # View the version of remote Kubernetes cluster

curl http://localhost:8001/ # View all the available APIs
```

Each resource under the API group has a set of actions (verbs) assigned to it like `create`, `delete`, `watch`, `list` etc.

```bash
# Example
curl http://localhost:8001/apis/apps/v1/

...
{
  "name": "deployments",
  "singularName": "",
  "namespaced": true,
  "kind": "Deployment",
  "verbs": [ # Set of actions that can be performed on a resource
    "create",
    "delete",
    "deletecollection",
    "get",
    "list",
    "patch",
    "update",
    "watch"
  ],
  "shortNames": [
    "deploy"
  ],
  "categories": [
    "all"
  ],
  "storageVersionHash": "8bacaeSe+gvE="
}
...
```

### API Versions

In Kubernetes and API can have different versions deployed at the same time, for example, v1alpha1, v2beta1, v1 etc.

|              | Alpha                                            | Beta                                                    | GA                                      |
| ------------ | ------------------------------------------------ | ------------------------------------------------------- | --------------------------------------- |
| Version Name | vXalphaY (e.g. v1alpha1)                         | vXbetaY (e.g. v1beta1)                                  | vX (e.g. V1)                            |
| Enabled      | No enabled by default                            | Enabled by default                                      | Enabled by default                      |
| Tests        | May lack end to end tests                        | Has end to end tests                                    | Has conformance tests                   |
| Reliability  | May have bugs                                    | May have minor bugs                                     | Highly reliable                         |
| Support      | No commitments. May be dropped later.            | Commits to complete the feature and move to GA          | Will be present in many future releases |
| Audience     | Expert users interested in giving early feedback | Users interested in beta testing and providing feedback | All users                               |

_Preferred version_ - When multiple versions of an API are enabled then preferred is the default version that is used when you retrieve information through that API irrespective of the version you have used in the YAML file when creating the object. There can be only one preferred version.

_Storage version_ - When multiple versions of an API are enabled then storage version is the default version in which the object is stored in etcd irrespective of the version you have used in the YAML file when creating the object. There can be only one storage version.

> Usually the preferred and storage versions are same, however, they can be different as well.

```bash
# To view the preferred version query the API
$ curl http://localhost:8001/apis/batch
{
  "kind": "APIGroup",
  "apiVersion": "v1",
  "name": "batch",
  "versions": [
    {
      "groupVersion": "batch/v1",
      "version": "v1"
    },
    {
      "groupVersion": "batch/v1beta1",
      "version": "v1beta1"
    }
  ],
  "preferredVersion": {
    "groupVersion": "batch/v1",
    "version": "v1"
  }
}

# As of now it is not possible to see which is the Storage version of an API through an API call or Kubectl command. One way to check it is by directly viewing the stored object in etcd.
```

To view API version, short names and other details like is the API namespaced or not, execute following command:

```bash
kubectl api-resources
```

### API Deprecation

API Deprecation policy:
Other than the most recent API versions in each track, older API versions must be supported after their announced deprecation for a duration of no less than:

- GA: 12 months or 3 releases (whichever is longer)
- Beta: 9 months or 3 releases (whichever is longer)
- Alpha: 0 releases

To convert Kubernetes YAML definition files from older version to newer version:

```bash
kubectl convert -f <old-file> --output-version <new-api>

# Example:
kubectl convert -f deployment.yaml --output-version apps/v1
```

## Authentication, Authorization and Admissions Control

### Kubernetes Security

Kubernetes API server is at the center of all our interactions with Kubernetes. Controlling access to the API server is the first line of defense where authentication can authorization comes into picture.

- Authentication (Who can access?) - There are several ways in which authentication can be implemented in Kubernetes like:
  - Static Files containing Username and Passwords
  - Static Files containing Username and Tokens
  - Certificates
  - Integration with external Identity Provider like Azure Active Directory
  - Service Accounts (for non-human interactions with api server)
- Authorization (What can they do?) - There are different types of authorizations supported by Kubernetes
  - RBAC (Role based access control)
  - ABAC (Attribute based access control)
  - Node Authorization
  - Webhook Mode

Storing usernames, passwords or tokens in a static file and using that as an authentication mechanism for kubernetes is not a recommended approach since it is insecure. Also note that this approach is deprecated in Kubernetes version 1.19 and is no longer available in later releases.
{: .notice--danger}

All communication between cluster components like `kube-apiserver`, `kubelet`, `etcd`, `scheduler`, `Controller Manager`, `kube proxy` is secured using TLS certificate encryption.

Communication between the pods can be secured by using Network Policies or more complex mechanisms like Service Mesh.

### Authentication

Types of users:

- _Admins_ who access the cluster to perform administrative tasks.
- _Developers_ who access the cluster to test or deploy applications.
- _End Users_ who access the applications deployed on the cluster.
- _Bots_ (or automated processes) which access the cluster for integration purposes.

Security of the end users who access the applications deployed on Kubernetes needs to be managed by the application internally.
{: .notice--info}

There are two types of accounts in Kubernetes:

1. User Account - Used by humans (Admins and Users) to perform certain tasks on Kubernetes cluster. For e.g. Admin performing administrative tasks on the Kubernetes cluster or a developer deploying an application on Kubernetes. Kubernetes does not manage user accounts natively, it relies on external identity providers like Azure Active Directory, Static Password file, Static Token file, certificates etc. to manage user identities. All user access to Kubernetes is managed by the Kube API Server.
2. Service Account - is used by an application to interact with Kubernetes cluster. For e.g. Prometheus uses service account to poll k8s API to fetch performance metrics. Kubernetes can create and manage service accounts natively.

### Kubeconfig File

Kubeconfig file is used to organize information about clusters, users, namespaces, and authentication mechanisms. When you run the `kubectl` commands, it automatically picks up the current context and authentication information from the Kubeconfig file and uses that information to authenticate with Kubernetes cluster and execute the command.

> Default path for KubeConfig file - $HOME/.kube/config
>
> If your Kubeconfig file is not present in the default location then you can pass the kubeconfig to the `kubectl` command like this:
>
> kubectl config view --kubeconfig=my-custom-config

Kubeconfig file contains three sections:

- Clusters - are various clusters that you need access to (like Dev, Prod, AKS, EKS etc.).
- Users - are the user accounts with which you have access to the various clusters (like Dev User, Prod User, Admin etc.).
- Context - defines which user account will be used to access which cluster.

<p align="center">
   <img alt="Kubernetes config" src="/assets/images/kubernetes-fundametals/kubeconfig.svg" style="max-height: 600px">
</p>

Format of Kubeconfig file:

```yaml
apiVersion: v1
kind: Config
clusters:
  - name: my-k8s-playground
    cluster:
      certificate-authority: ca.cert
      server: https://my-k8s-playground:6443
contexts:
  - name: my-k8s-admin@my-k8s-playground
    context:
      cluster: my-k8s-playground
      user: my-k8s-admin
users:
  - name: my-k8s-admin
    user:
      client-certificate: admin.crt
      client-key: admin.key
current-context: my-k8s-admin@my-k8s-playground
```

### Authorization

Control what actions can a user or an application can perform in the cluster. We should follow the principle of least privileges to ensure that a user or an application has minimum level of access necessary to perform their task.
For example:

- A developer might be allowed to deployed new pods but not allowed to delete a namespace or a service.
- When working in different teams, Team A should be only allowed to access namespace A and Team B should be only allowed to access namespace B.

There are different authorization mechanisms:

- _Node Authorization_ is a special-purpose authorization mode that specifically authorizes API requests made by kubelet.
- _ABAC (Attribute based access control)_ is where you associate a user or a group of users with a set of permissions like Can view Pods, Can create Pods, Can delete Pods. This is accomplished with the help of policy files. However, every time you need to add or make a change in security, you must edit the policy file manually and restart the Kube API Server. As such ABAC based configuration are difficult to manage.
- _RBAC (Role based access control)_ - With RBAC, instead of directly associating a user or a group with a set of permissions, we define roles, for example, a Developer Role. We create this role with the set of permissions required for developers. Then we associate all the developers to that role.
  Similarly, we can create a role for Security users with the right set of permissions required for them. Then associate the user to that role. Going forward whenever a change needs to be made to the users access, we simply modify the role and it reflects on all users belonging to that role immediately.
  Role based access control provides a more standard approach to managing access within the Kubernetes cluster.
- _Webhook Mode_ is used to outsource the authorization mechanism to an external provider like Open Policy Agent. Here the API Server makes a call to the external provider and the external provider determines if a user is authorized to perform a certain action or not.
- _Always Allow_ - Allow all requests without performing any authorization.
- _Always Deny_ - Deny all requests.

You can configure more than one authorization modes at a time:

```bash
--authorization-mode=Node,RBAC,Webhook # In this case the authorization request is evaluated by using authorization mechanism from left to right i.e. The request will be first evaluated by using Node authorization, if that fails then the same request will be evaluated by using RBAC. If RBAC succeed then no further evaluation is done.
```

How to check which authorization modes are configured on the API Server:

```bash
kubectl describe pods kube-apiserver-controlplane -n kube-system
```

### Role Based Access Control (RBAC)

In Kubernetes we can define two types of accounts - User Accounts and Service Accounts. But how do we define which user/service account has what level of access in Kubernetes? For this Kubernetes allows us to create _Roles_ and _Cluster Roles_. Using Roles we can define who has what level of access on which Kubernetes resources.

It is important to note that different resources in Kubernetes have different scope. Some resources are available at the namespace level whereas other resources are available at the cluster level. To get a full list of namespaced and non-namespaced resources use following commands:

```bash
kubectl api-resources --namespaced=true # To get list of resources where scope is namespace

kubectl api-resources --namespaced=false # To get list of resources which are at cluster scope
```

_Roles_ and _Role bindings_ fall under the scope of a namespace which means these are created within particular namespace. However, when you need to authorize the users to cluster wide resources like nodes, persistent volumes etc. we use _Cluster Roles_ and _Cluster Role Bindings_. For example, a cluster admin role can be created to provide an administrator permissions to view, create or delete nodes in a cluster. Similarly, a storage administrator role can be created to authorize a storage admin to create persistent volumes.

How to check what access you have as a user on the Kubernetes cluster?

```bash
kubectl auth can-i create deployments

kubectl auth can-i delete pods
```

If you are an Admin, you can also impersonate a user and check what access they have on the Kubernetes Cluster:

```bash
kubectl auth can-i create deployments --as dev-user

kubectl auth can-i delete configmaps --as dev-user

kubectl get pod <Pod name> --as dev-user

kubectl create deployment <deployment name> --as dev-user
```

#### Roles and Role bindings

The _first step_ is to define a Role, like a Developer Role:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
  - apiGroups: [""] # Leave blank if you are specifying rule for Core API Group, else if you are defining the rule for Named group then specify the name of the API group
    resources: ["pods"]
    verbs: ["list", "get", "create", "update", "delete"]
    resourceNames: ["frontend", "database"] # This field is used for providing access to specific resources. In this case we are allowing the user to access only frontend and database pods.
  - apiGroups: [""]
    resources: ["ConfigMaps"]
    verbs: ["create"]
```

The _second step_ is to bind the role to a user:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-rolebinding
subjects: # Define the users
  - kind: User
    name: dev-user
    apiGroup: rbac.authorization.k8s.io
roleRef: # Defines the role that needs to bound to the user
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

#### Cluster Roles and Cluster Role bindings

Defining a Cluster Role is very similar to defining a normal Role:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
  - apiGroup: []
    resources: ["nodes"]
    verbs: ["create", "delete", "get"]
```

To link the user to the cluster role we create Cluster Role Binding object:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
  - kind: User
    name: cluster-admin
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

Cluster Roles and Cluster Role bindings can also be used for namespace scoped resources like Pods, Deployments etc. When a user is assigned a Cluster Role, they will have access to these resources across all namespaces.
{: .notice--info}

### Admissions Controller

When you make a new request to the Kubernetes cluster, let's say for creating a new pod, the request goes to the API Server where it is first authenticated and then authorized to make sure that you have the right set of permissions to perform the required action. If not, the request is reject by the API Server. If you have the right permissions then you are allowed to create the pod and the information is persisted in the etcd database.

<p>
  <img alt="Kubernetes API server authentication and authorization" src="/assets/images/kubernetes-fundametals/apiserver.svg">
</p>
For authentication and authorization we make use of Kubernetes RBAC (Roles, ClusterRoles). With RBAC you can define which user has what access on a certain Kubernetes object but you cannot define anything beyond that. For example, with RBAC you cannot define rules like _Allow pulling images from specific registries_ or _Reject deployments that do not define resource limits on the pods_ etc.

This is where **Admission Controllers** come into picture. Simply put Admission Controllers are plugins that govern and enforce how the Kubernetes cluster is used. They can be thought of as a gatekeeper that intercept (authenticated) API requests and may change the request object or deny the request altogether.

For example, the `LimitRanger` admission controller can augment pods with default resource requests and limits (mutating phase), as well as verify that pods with explicitly set resource requirements do not exceed the per-namespace limits specified in the LimitRange object (validating phase), Or When a namespace is deleted and subsequently enters the `Terminating` state, the `NamespaceLifecycle` admission controller is what prevents any new objects from being created in this namespace.

There are many different admission controllers that come pre-built into Kubernetes:

- AlwaysPullImages
- DefaultIngressClass
- DefaultStorageClass
- EventRateLimit
- NamespaceLifecycle

> More details about different types of pre-built Admission Controllers can be found [here](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#what-does-each-admission-controller-do).

#### Why do I need admission controllers?

- _Security_: Admission controllers can increase security by mandating a reasonable security baseline across an entire namespace or cluster. The built-in `PodSecurityPolicy` admission controller is perhaps the most prominent example; it can be used for disallowing containers from running as root or making sure the container’s root filesystem is always mounted read-only, for example. Further use cases that can be realized by custom, webhook-based admission controllers include:
  - Allow pulling images only from specific registries known to the enterprise, while denying unknown image registries.
  - Reject deployments that do not meet security standards. For example, containers using the `privileged` flag can circumvent a lot of security checks. This risk could be mitigated by a webhook-based admission controller that either rejects such deployments (validating) or overrides the `privileged` flag, setting it to `false`.
- _Governance_: Admission controllers allow you to enforce the adherence to certain practices such as having good labels, annotations, resource limits, or other settings. Some of the common scenarios include:
  - Enforce label validation on different objects to ensure proper labels are being used for various objects, such as every object being assigned to a team or project, or every deployment specifying an app label.
  - Automatically add annotations to objects, such as attributing the correct cost center for a “dev” deployment resource.
- _Configuration management_: Admission controllers allow you to validate the configuration of the objects running in the cluster and prevent any obvious misconfigurations from hitting your cluster. Admission controllers can be useful in detecting and fixing images deployed without semantic tags, such as by:
  - automatically adding resource limits or validating resource limits
  - ensuring reasonable labels are added to pods, or
  - ensuring image references used in production deployments are not using the latest tags, or tags with a -dev suffix.

#### Types of Admission Controllers

There are two types of admission controllers:

1. Mutating Admission Controllers - Can mutate or modify the incoming request. E.g. `DefaultStorageClass` Admission Controller - If the user does not specify a storage class when creating a new PVC then DefaultStorageClass admission controller automatically sets the default storage class by modifying the PVC creation request.
2. Validating Admission Controllers - Can validate the incoming request. E.g. `NamespaceExists` Admission controller will validate if the namespace already exists or not before allowing the request to proceed.

The admission control process has two phases: the mutating phase is executed first, followed by the validating phase. Consequently, admission controllers can act as mutating or validating controllers or as a combination of both. For example, the LimitRanger admission controller can augment pods with default resource requests and limits (mutating phase), as well as verify that pods with explicitly set resource requirements do not exceed the per-namespace limits specified in the LimitRange object (validating phase).

#### Custom/External Admission Controllers

Apart from the Admission Controllers shipped along with Kubernetes, it also allows you to create your own custom admission controllers with your own logic for mutations and validations or add an external admission controllers. This is made possible via two special admission controllers - `MutatingAdmissionWebhook` and `ValidatingAdmissionWebhook`.

<p>
<img alt="Custom admissions controller" src="/assets/images/kubernetes-fundametals/webhookcontroller.svg"/>
</p>
The Admission controller server can be deployed inside or outside of Kubernetes.

## Custom Resource Definition (CRD), Custom Controllers and Operators

When we create a new resource object in Kubernetes like a Pod or a deployment, all it does is store that configuration in etcd data store. When we list, modify or delete the resource object, all it does is list, modify or delete the object in the etcd data store. However, we know that when we create a new deployment with x number of replicas, Kubernetes creates that deployment and runs x number of replicas for a Pod. So who or what is responsible for doing this?

This is where the `Controller` comes into picture (e.g. Deployment Controller). A controller is a process that runs in the background and its job is to continuously monitor the status of the resources that it is supposed to manage like a deployment. So, when we create, update or delete the deployment, the Deployment Controller makes the necessary changes on the cluster to match the action that we executed. In short, a Controller contains the actual business logic to create a deployment, replicas, pods etc. on the Kubernetes cluster. So, that is why we generally refer to controller as the brains of Kubernetes.
Example:

- ReplicaSet resource is accompanied by a ReplicaSet Controller
- Deployment resource is accompanied by a Deployment Controller
- Namespace resource is accompanied by a Namespace Controller
- Job resource is accompanied by a Job Controller

### Custom Resource Definition (CRD) and Custom Controllers

In case you need to create any resource apart from the already available resources (like deployments, pods, replicas, jobs etc.), you need to create a Custom Resource Definition (CRD).

When you create a new `CustomResourceDefinition` (CRD), the Kubernetes API Server creates a new RESTful resource path for each version you specify. The custom resource created from a CRD object can be either namespaced or cluster-scoped. Once the custom resource is registered, end users can create, update and delete its object using kubectl commands, similar to how users interact with built-in resources, like pods, deployments and services.

```yaml
# Example CRD:

apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: learncontainers.theteckie.com # name must match the spec fields below, and be in the form: <plural>.<group>
spec:
  scope: Namespaced # either Namespaced or Cluster
  group: theteckie.com # group name to use for REST API: /apis/<group>/<version>
  names:
    kind: LearnContainers # This will be used in the resource manifest file
    singular: learncontainer
    plural: learncontainers # plural name to be used in the URL: /apis/<group>/<version>/<plural>
    shortNames:
      - lc
  versions:
    - name: v1
      served: true # Enable/Disable if this version will be served by the API Server.
      storage: true # One and only one version must be marked as the storage version.
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                courseName:
                  type: string
                courseId:
                  type: integer

# To deploy this CRD
kubectl create -f customcrd.yaml

# Create a LearnContainers resource using YAML definition file
apiVersion: theteckie.com/v1
kind: LearnContainers
metadata:
  name: learncontainers-crd
spec:
  courseName: "Containers-101"
  courseId: 22

# Deploy custom resource:
kubectl create -f learncontainers.yaml

# Get custom resource:
kubectl get LearnContainers
kubectl explain LearnContainers
kubectl describe lc learncontainers-crd

```

CRDs do not have any logic attached, nor any special behavior by default; once they are created, modified or removed, they take no actions on their own. This is where the Custom Controllers come into picture. Custom Controllers implement the required business logic which should be executed when a CRD object is created, updated or deleted from the cluster.

For the sample CRD above we will need to create a Custom Controller which monitors etcd data store for `LearnContainers` custom resource and executes the business logic like adding the specific course to a learning plan when a new LearnContainers resource is created.

```go
// Sample controller code outline in Go language
package learncontainers

var controllerkind = apps.SchemeGroupVersion.WithKind("LearnContainers")

// <Code hidden>

// Run beings watching and syncing.
func (dc *LearnContainersController) Run(workers int, stopCh <-chan struct{})

// <Code hidden>
// Call the external service to add course to the learning path
func (dc *LearnContainersController) callLearningPathsAPI(obj interface{})

// <Code hidden>
```

The custom controller can be written in any language or framework that you are comfortable with. Many languages like Go also come with Kubernetes Client Libraries which makes the development of custom controller easier.
Sample code for a custom controller can be found [here](#https://github.com/kubernetes/sample-controller).

The custom controller can be deployed inside or outside of Kubernetes cluster as required.

### Operator Framework

CRDs and Custom Controllers can be packaged together to be deployed and managed as a single entity using the Operator Framework.
All operators are available at the [OperatorHub.io](#https://operatorhub.io/). Here you can find operators for many popular applications like MySQL, Prometheus, Grafana, etcd etc.

## Helm

Applications that we deploy in our Kubernetes cluster can become very complex with multiple objects which needs to interconnect with each other to make the application work.

For example: A Wordpress deployment on Kubernetes may require creation of multiple objects like a deployment for declaring the pods, mysql server, web server, etc., a Persistent volume along with a Persistent Volume claim to store the database, a service to expose the Wordpress site to internet, Kubernetes secret to store username and password etc. For each Object we will declare a separate YAML file and then to deploy the application we will do kubectl apply for each YAML file. Now imagine that we need to change something in the application like change the amount of storage for a persistent volume. We need to look through multiple YAML files and make sure that we are making the changes very carefully. Or let's say we need to upgrade a certain component of the application or we may need to delete the complete application then we need to keep track of all the different components which belong to the application and delete them one by one.

The problem here is that Kubernetes does not understand that all these different objects are part of a single application and these components need to be installed or removed together. All Kubernetes knows that these are different objects like Pods, Deployments, Secrets, and it just proceeds to deploy these objects and make them available for us.

This is where Helm comes into picture. Helm is built to know about such stuff, it looks at all these different objects as part of a big package. That’s why it is also known as Package Manager for Kubernetes.

Let’s understand this through an example – Let’s say you want to install and run a video game on your computer. A video game consists of lots of different components – executable code files, audio and video files, game graphics files, textures, images, configuration, data and so on. Now imagine if you have to install each of these files separately, that would be very tedious and error prone. Fortunately, the game developers’ package everything up into a game installer which we download and run on our PC. We just select in which directory on our PC the game should be installed and then press the install button and the game installer does the rest. When we want to uninstall the game, we just use the uninstall feature and everything gets automatically removed from our PC without us needing to keep track of different game files and assets.

Helm works on similar lines. When we have a complex application with 100s of Kubernetes objects we can package the application using something known as `Helm Chart`, then we can deploy the complete application by just using a single command. We can also customize the deployment with different values and settings as required through a single file called `values.yaml`.

### Helm Charts

Helm uses a packaging format called charts. A chart consists of three important components – A templates folder (this is where our YAML definition files will be placed), a `values.yaml` file and a `chart.yaml` file.
The hard coded values in the YAML definition files can be replaced by variables from the `values.yaml` file.

```bash
# Sample directory structure for helm chart for Wordpress
wordpress/
  Chart.yaml          # A YAML file containing information about the chart
  LICENSE             # OPTIONAL: A plain text file containing the license for the chart
  README.md           # OPTIONAL: A human-readable README file
  values.yaml         # The default configuration values for this chart
  values.schema.json  # OPTIONAL: A JSON Schema for imposing a structure on the values.yaml file
  charts/             # A directory containing any charts upon which this chart depends.
  crds/               # Custom Resource Definitions
  templates/          # A directory of templates that, when combined with values,
                      # will generate valid Kubernetes manifest files.
  templates/NOTES.txt # OPTIONAL: A plain text file containing short usage notes
```

More information about Helm Charts can be found [here](#https://helm.sh/docs/topics/charts/).

> You can find 1000s for existing helm chart at [artifacts.io](https://artifacthub.io/).

```bash
helm install [release-name] [chart-name] # Install helm chart

helm list # List installed packages

helm uninstall [release-name] # Uninstall helm chart

helm search hub [chart-name] # To search for existing helm charts in Artifact Hub

helm repo add [repository-name] [repository-url] # Add a repository to helm

helm search repo [chart-name] # To search the chart in repository other than hub

helm repo list # List existing repositories added to helm

helm pull --untar [chart-name] # Pull the helm package and untar the file

helm show values [chart-name] # View the values from values.yaml file

helm install [release-name] [chart-name] --set replicaCount=3 # Set values when installing new release
```

### References

- [Kubernetes documentation - API Groups](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/)
- [Kubernetes documentation - API Versioning](https://kubernetes.io/docs/reference/using-api/)
- [Kubernetes documentation - API Deprecation](https://kubernetes.io/docs/reference/using-api/deprecation-policy/)
