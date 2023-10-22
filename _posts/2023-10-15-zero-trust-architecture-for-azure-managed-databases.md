---
title: "Zero Trust Architecture for Azure Managed Databases"
date: 2023-10-15
excerpt: ""
header:
  teaser: "https://raw.githubusercontent.com/shivkanwer/shivkanwer.github.io/main/assets/images/zero-trust-arch/zero-trust-arch-for-azure-databases.jpg"
classes: wide
categories:
  - Architecture
tags:
  - cloud
  - azure
  - security
  - zero-trust
---

- [Network Isolation](#network-isolation)
- [Access Control](#access-control)
  - [User Access](#user-access)
  - [Workload Access](#workload-access)
  - [DevOps Access](#devops-access)
- [Data Protection](#data-protection)
- [Auditing](#auditing)
- [References](#references)

<img alt="Zero Trust Architecture diagram for Azure Managed Databases" src="/assets/images/zero-trust-arch/zero-trust-arch-for-azure-databases.svg">

## Network Isolation

## Access Control
One of the most important aspects of Zero Trust architecture is Access Control. Integrate Azure Managed database with Microsoft Entra ID and disable direct database username and password access. Any user or workload must connect with the database using their own identity assigned via Microsoft Entra ID. This allows you to be free from storing database credentials in config files, Key vault, Kubernetes secrets etc. and instantly reduces the security risks related with database password leakage. 

### User Access
Define security groups in Microsoft Entra ID for different personas which are expected to access the database, for e.g., Database Administrators, Database Developers, Database Support etc. Map the groups to appropriate roles like admin, writer, reader etc in the database. Enable Privileged Identity Management (PIM) for the security groups defined earlier. This allows you to grant users just-in-time membership to these groups. Any time a user needs to access the database, they must raise a request through PIM which goes through an approval process. Once approved, the user is granted temporary time-bound access to the designated Microsoft Entra ID group which is further mapped to appropriate role in the database. Now the user can log into the database using their own identity and perform their tasks.

Since the groups are enrolled under PIM, you can define polices like time-bound access, require justification, approval process, multi-factor authentication, conditional access etc.   

<p align="left">
  <img alt="Privileged Identity Management options screenshot" src="/assets/images/zero-trust-arch/pim-settings.jpg" style="max-width: 350px;border: 1px solid black;"/>
</p>

### Workload Access 
To enable workloads (such as an application, service, script, or container) to interact with Azure Managed databases, Azure provides the feature of *Workload Identity*. A workload identity is an identity you assign to a software workload to authenticate and access other services and resources. Azure supports three types of workload identities - *Service Principals*, *Managed Identity* and *Application*[^1]. It is recommended to use Managed Identity where possible instead of a Service Principal. With Service Principal you are responsible for managing and rotating secrets, whereas with Managed Identity, Microsoft Entra ID automatically manages and rotates the secrets for you. Additionally, the secrets are not exposed by Managed Identity, the application can make use of Azure.Identity Library[^2] to consume the secrets at runtime.  

[^1]: [Microsoft Entra ID - Workload Identity](https://learn.microsoft.com/en-us/azure/active-directory/workload-identities/workload-identities-overview)
[^2]: [Azure Identity Client Libraries](https://learn.microsoft.com/en-us/azure/aks/workload-identity-overview?tabs=dotnet#azure-identity-client-libraries)

Following is an example architecture where an application running on Azure Kubernetes Service (AKS) makes use of Workload Identity to authenticate and connect with Azure MySQL database:  

<img alt="Connect with Azure MySQL Database using Azure Kubernetes Service (AKS) Workload Identity architecture diagram" src="/assets/images/zero-trust-arch/aks-workload-identity.svg">

1. Enable OIDC issuer feature on the AKS cluster. This feature enables the AKS cluster to act as an OIDC token issuer.
2. Enable Workload Identity feature of the AKS cluster. This feature deploys a mutating webhook admissions controller on the AKS cluster which injects the pod with workload identity details during deployment.
3. Create a new Service Account which is linked to the Azure Managed Identity through client ID.
4. Establish a trust relationship between Azure Managed identity and the AKS OIDC token issuer and Service Account. Since the Managed Identity is managed via Microsoft Entra ID, there is already a trust relationship between Managed Identity and Microsoft Entra ID.
5. During deployment configure the application pod to use the Service Account defined in Step 3. The Workload Identity Webhook injects the identity details into the container as environment variables. It also projects the signed federated access token from AKS OIDC token issuer into the local path inside the container.
6. Application code makes request for connecting with the database using the signed access token. Microsft Entra ID validates the token from application and in exchange provides the Entra ID access token which can then be used by the application to connect with the database. The token exchange is facilitated by the Azure Identity Library. 

### DevOps Access

## Data Protection

## Auditing

## References
