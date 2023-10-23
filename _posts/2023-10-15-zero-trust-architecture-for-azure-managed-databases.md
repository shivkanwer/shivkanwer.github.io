---
title: "Zero Trust Architecture for Azure Managed Databases"
date: 2023-10-15
excerpt: "In this blog post, I delve into the intricacies of constructing a Zero Trust architecture tailored explicitly for Azure Managed Databases. While Zero Trust is a multifaceted concept, the primary focus of this blog centers on its application within the Azure managed database landscape."
header:
  teaser: "/assets/images/zero-trust-arch/zero-trust-arch-for-azure-databases.jpg"
classes: wide
categories:
  - Architecture
tags:
  - cloud
  - azure
  - security
  - zero-trust
---
In this blog post, I delve into the intricacies of constructing a Zero Trust architecture tailored explicitly for Azure Managed Databases. While Zero Trust is a multifaceted concept, the primary focus of this blog centers on its application within the Azure managed database landscape.

## Zero Trust Model
The Zero Trust model[^1] emphasizes the continuous validation of trust rather than making any assumptions about it. In the past, when all users, applications, devices, and data resided within an organization's firewall, they were automatically considered trusted. This pre-established trust often led to the ease of lateral movement for malicious actors once they compromised an endpoint device.

Today, a majority of users access applications and data from the internet, and many organizations permit the use of personal devices for work. Key elements of a transaction, such as users, networks, and devices, are no longer entirely within the organization's control. The Zero Trust model operates on the assumption of a breach and mandates the verification of each request as if it were originating from an open network. Irrespective of the request's source or the resource it seeks to access, Zero Trust instills the principle of *'never trust, always verify.'*

[^1]: [Zero Trust Model - Microsoft](https://www.microsoft.com/en-in/security/business/zero-trust)

## Architecture
<img alt="Zero Trust Architecture diagram for Azure Managed Databases" src="/assets/images/zero-trust-arch/zero-trust-arch-for-azure-databases.svg">

*Download a [draw.io](https://contentstore101.blob.core.windows.net/thetecktalk/zero-trust-arch-azure-databases.drawio?sp=r&st=2023-10-23T08:20:30Z&se=2043-10-23T16:20:30Z&spr=https&sv=2022-11-02&sr=b&sig=jC%2FxMT4ODPA2%2Bj3yowRnMovWOcNXV8qwJ3yGkJorYik%3D) file of this architecture.*

## Network Isolation
Azure managed databases provide a range of network isolation features, including Private Endpoints, VNET Integration, Firewall rules, and the ability to disable internet access. The underlying concept is to adopt a Zero Trust principal of *assume breach* and, as a result, limiting the potential impact by blocking direct internet access and permitting traffic exclusively from specified VNETs and IP ranges.

In the architecture diagram provided above, the Private Endpoint[^2] feature is used to incorporate the database into a designated subnet within a Virtual Network (VNET). It's important to note that direct public network access is intentionally turned off. Furthermore, at the subnet level, a Network Security Group (NSG) has been configured to function as a firewall, granting you the capability to manage both incoming and outgoing network traffic for that specific subnet. 

[^2]: [Private Endpoint](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-overview)

## Access Control

Access Control is one of the most important pillars of a Zero Trust architecture. When implementing Zero Trust for Azure Managed Databases, integrate the database with Microsoft Entra ID and disable the use of native database username and password. This implementation mandates that all user and workload must connect with the database using their own identity assigned through Microsoft Entra ID. This approach liberates you from the necessity of storing database credentials in various locations like config files, Key Vault, or Kubernetes secrets, thereby instantaneously mitigating security risks associated with potential database password exposure.

#### User Access
When it comes to user access to the database, a fundamental principle to adhere to is 'Keep people away from data'. It's crucial not to provide users with continuous 24/7, unrestricted access to the database. All requests for database access must undergo a thorough review and approval process.

To implement this, create distinct security groups within Microsoft Entra ID for various user roles that require access to the database. These roles might include Database Administrators, Database Developers, Database Support, and more. Each of these security groups is associated with specific roles like Admin, Writer, Reader etc. within the database, following the principle of *'least-privilege access'*.

Utilize Azure's Privileged Identity Management (PIM) to manage the defined security groups[^3]. PIM enables the provisioning of just-in-time memberships to these groups. Whenever a user requires database access, they must initiate a request through PIM, which follows an approval workflow. Once approved, the user is granted temporary, time-bound access to the designated Microsoft Entra ID group, which is further mapped to an appropriate role within the database. Subsequently, the user can log into the database using their own identity and perform their tasks. Once the membership expires, the user is automatically removed from the security group and they no longer have access to the database.

[^3]: [Privileged Identity Management (PIM) for Groups](https://learn.microsoft.com/en-us/azure/active-directory/privileged-identity-management/concept-pim-for-groups)

Given that these groups are managed under PIM, you have the flexibility to establish policies such as time-bound access, mandatory justification, approval workflows, multi-factor authentication, conditional access, and more.
<p align="left">
  <img alt="Privileged Identity Management options screenshot" src="/assets/images/zero-trust-arch/pim-settings.jpg" style="max-width: 400px;border: 1px solid black;"/>
</p>

**Trade-off**

#### Workload Access 
In the context of enabling workloads, including applications, services, scripts, and containers, to interact with Azure Managed databases, Azure introduces the concept of *Workload Identity*. Workload identity represents a distinct identity assigned to the software workload, enabling it to securely authenticate and access various services and resources. Azure offers three primary types of workload identities: Service Principals, Managed Identity, and Application[^4].

While these options have their merits, it is highly recommended to utilize Managed Identity whenever feasible, instead of opting for a Service Principal. The key distinction lies in how secrets are managed. When using a Service Principal, the responsibility of managing and regularly rotating secrets rests with you. In contrast, Managed Identity takes a significant burden off your shoulders as Microsoft Entra ID automatically handles the management and rotation of secrets on your behalf. Furthermore, Managed Identity ensures the secrecy of these secrets and allows your application to dynamically retrieve and utilize them at runtime through the use of Azure Identity Client Libraries[^5] and the Microsoft Authentication Library (MSAL)[^6]. 

Additionally, it is essential to associate the Workload Identity with a specific role in the database, adhering to the principle of 'least-privilege access'. This further refines your security posture by granting only the minimum necessary permissions for the workload to perform its tasks.

[^4]: [Microsoft Entra ID - Workload Identity](https://learn.microsoft.com/en-us/azure/active-directory/workload-identities/workload-identities-overview)
[^5]: [Azure Identity Client Libraries](https://learn.microsoft.com/en-us/azure/aks/workload-identity-overview?tabs=dotnet#azure-identity-client-libraries)
[^6]: [Microsoft Authentication Libraries](https://learn.microsoft.com/en-us/azure/active-directory/develop/msal-overview)

Following is an example architecture where an application running on Azure Kubernetes Service (AKS) makes use of Workload Identity to authenticate and connect with Azure MySQL database:  

<img alt="Connect with Azure MySQL Database using Azure Kubernetes Service (AKS) Workload Identity architecture diagram" src="/assets/images/zero-trust-arch/aks-workload-identity.svg">


In this example, Azure MySQL database is integrated with Microsoft Entra ID and an Azure Managed Identity is setup to 
- The OIDC feature lets the AKS cluster act as an token issuer, issuing tokens to the Kubernetes Service Accounts. 
- The Workload Identity feature deploys a mutating webhook admissions controller which injects identity details in the pod.
Further, the Azure Managed Identity is configured to trust the access tokens issued by AKS. At runtime, the workload can exchange the service account token projected to its volume for an Entra ID access token via the use of Azure Identity Library and then use that token to connect with the database. 

## Data Protection
- Azure Managed Databases support data encryption at rest by using Azure managed key or customer-managed key. With customer-managed key, you fully control data access by the ability to remove the key and make the database inaccessible. You have full control over they key lifecycle, including rotation of key to align with corporate policies. This also allows you to implement separation of duties for managing keys and data. 
- Azure Managed Databases use SSL/TLS to encrypt the data in transit. 

## Auditing
Database auditing is a critical component of data security, compliance and integrity. Auditing enables the detection of unauthorized actions, as well as the actions performed by authorized users. It reveals who did what, and what was affected. Many industries and regions have strict regulations that mandate auditing of database activities, for e.g. HIPAA, GDPR etc. 

All Azure Managed Databases support database auditing features. Once turned on, the audit logs can be send to Azure Log Analytics. For long term storage requirements the audit logs can be exported to Azure Storage account. 

## Compliance

## References
