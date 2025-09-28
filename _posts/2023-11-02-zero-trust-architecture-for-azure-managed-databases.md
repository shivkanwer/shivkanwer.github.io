---
title: "Zero Trust Architecture for Azure Managed Databases"
date: 2023-11-02
excerpt: "In this blog post, I delve into the intricacies of constructing a Zero Trust architecture tailored explicitly for Azure Managed Databases. While Zero Trust is a multifaceted concept, the primary focus of this blog centers on its application within the Azure managed database landscape."
header:
  teaser: "https://raw.githubusercontent.com/shivkanwer/shivkanwer.github.io/main/assets/images/zero-trust-arch/zero-trust-arch-for-azure-databases.jpg"
layout: wide-no-author
categories:
  - Architecture
tags:
  - cloud
  - azure
  - security
  - zero-trust
---
In this blog post, I delve into the intricacies of constructing a Zero Trust architecture tailored explicitly for Azure Managed Databases. While Zero Trust is a multifaceted concept, the primary focus of this blog centers on its application within the Azure managed database landscape.

**In this article**
- [Zero Trust Model](#zero-trust-model)
- [Architecture](#architecture)
- [Network Isolation](#network-isolation)
- [Access Control](#access-control)
  - [User Access](#user-access)
  - [Workload Access](#workload-access)
- [Data Protection](#data-protection)
- [Auditing](#auditing)
- [Compliance](#compliance)
- [References](#references)

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

### User Access
When it comes to user access to the database, a fundamental principle to adhere to is *'Keep people away from data'*. It's crucial not to provide users with continuous 24/7, unrestricted access to the database. All requests for database access must undergo a thorough review and approval process.

To implement this, create distinct security groups within Microsoft Entra ID for various user roles that require access to the database. These roles might include Database Administrators, Database Developers, Database Support, and more. Each of these security groups is associated with specific roles like Admin, Writer, Reader etc. within the database, following the principle of *'least-privilege access'*.

Utilize Azure's Privileged Identity Management (PIM) to manage the defined security groups[^3]. PIM enables the provisioning of just-in-time memberships to these groups. Whenever a user requires database access, they must initiate a request through PIM, which follows an approval workflow. Once approved, the user is granted temporary, time-bound access to the designated Microsoft Entra ID group, which is further mapped to an appropriate role within the database. Subsequently, the user can log into the database using their own identity and perform their tasks. Once the membership expires, the user is automatically removed from the security group and they no longer have access to the database.

[^3]: [Privileged Identity Management (PIM) for Groups](https://learn.microsoft.com/en-us/azure/active-directory/privileged-identity-management/concept-pim-for-groups)

Given that these groups are managed under PIM, you have the flexibility to establish policies such as time-bound access, mandatory justification, approval workflows, multi-factor authentication, conditional access, and more.
<p align="left">
  <img alt="Privileged Identity Management options screenshot" src="/assets/images/zero-trust-arch/pim-settings.jpg" style="max-width: 400px;border: 1px solid black;"/>
</p>

### Workload Access 
In the context of enabling workloads, including applications, services, scripts, and containers, to interact with Azure Managed databases, Azure introduces the concept of *Workload Identity*. Workload identity represents a distinct identity assigned to the software workload, enabling it to securely authenticate and access various services and resources. Azure offers three primary types of workload identities: Service Principals, Managed Identity, and Application[^4].

While these options have their merits, it is highly recommended to utilize Managed Identity whenever feasible, instead of opting for a Service Principal. The key distinction lies in how secrets are managed. When using a Service Principal, the responsibility of managing and regularly rotating secrets rests with you. In contrast, Managed Identity takes a significant burden off your shoulders as Microsoft Entra ID automatically handles the management and rotation of secrets on your behalf. Furthermore, Managed Identity ensures the secrecy of these secrets and allows your application to dynamically retrieve and utilize them at runtime through the use of Azure Identity Client Libraries[^5] and the Microsoft Authentication Library (MSAL)[^6]. 

Additionally, it is essential to associate the Workload Identity with a specific role in the database, adhering to the principle of *'least-privilege access'*. This further refines your security posture by granting only the minimum necessary permissions for the workload to perform its tasks.

[^4]: [Microsoft Entra ID - Workload Identity](https://learn.microsoft.com/en-us/azure/active-directory/workload-identities/workload-identities-overview)
[^5]: [Azure Identity Client Libraries](https://learn.microsoft.com/en-us/azure/aks/workload-identity-overview?tabs=dotnet#azure-identity-client-libraries)
[^6]: [Microsoft Authentication Libraries](https://learn.microsoft.com/en-us/azure/active-directory/develop/msal-overview)

Following is an example architecture where an application running on Azure Kubernetes Service (AKS) makes use of Workload Identity to authenticate and connect with Azure MySQL database:  

<img alt="Connect with Azure MySQL Database using Azure Kubernetes Service (AKS) Workload Identity architecture diagram" src="/assets/images/zero-trust-arch/aks-workload-identity.svg">

In this scenario, we establish an integration between an Azure MySQL database and Microsoft Entra ID and add Azure Managed Identity a user in database. We configure an Azure Kubernetes Service (AKS) cluster with OpenID Connect (OIDC) and Workload Identity features. With the OIDC feature enabled, the AKS cluster takes on the role of a token issuer, generating tokens for the Kubernetes Service Accounts. Meanwhile, the Workload Identity feature introduces a mutating webhook admissions controller, which automatically injects identity details into the pods. Further, we set up the Azure Managed Identity to trust the access tokens issued by AKS. During runtime, our workloads can exchange the service account token for an Entra ID access token. This token exchange is seamlessly executed through the Azure Identity Library. The workload can then use the Entra ID access token to securely connect with the Azure MySQL database.

More details about using Workload Identity with AKS can be looked up [here](https://learn.microsoft.com/en-us/azure/aks/workload-identity-overview?tabs=dotnet).

## Data Protection
**Data encryption at Rest and in Transit** 

Azure Managed Databases provide robust security through the encryption of data, both at rest and in transit. When it comes to data at rest, you have the flexibility to choose between Azure-managed keys or customer-managed keys. By opting for the latter, you gain full control over data access, including the ability to revoke the key, rendering the database inaccessible. This level of control extends to key lifecycle management, enabling you to align key rotation with corporate policies and implement separation of duties between key and data management. Furthermore, for data in transit, Azure Managed Databases utilize SSL/TLS encryption, ensuring the protection of your data as it travels between your application and the database.

**Backup and Disaster Recovery**

Database backups play a critical role in any comprehensive business continuity and disaster recovery plan as they serve as a safeguard against data corruption, security breaches, or accidental deletions. Azure Managed Databases provide an automated backup feature that simplifies this process. These backups are accessible for retention periods ranging from 7 to 35 days, granting you the flexibility to restore your database server to any specific point within that window. In a scenario where a security incident like ransomware attack happens, you have the flexibility to restore the database to a known good state in the past.

For organizations with data protection policies requiring extended backup availability, Azure offers the option to configure long-term retention (LTR)[^7] policies on a per-database basis.

Moreover, in the provided architectural diagram, in addition to the inherent point-in-time restoration capabilities, database backups are securely stored in geo-redundant storage within a paired Azure region. This redundancy helps with disaster recovery situation in the event where the primary region is down.

[^7]: [Long-term data retention](https://learn.microsoft.com/en-us/azure/azure-sql/database/long-term-retention-overview?view=azuresql)

**Threat Protection**

Azure provides a powerful tool, Microsoft Defender for Databases, which is a component of the Microsoft Defender for Cloud[^8] suite. It helps in identifying and flagging abnormal patterns in database access, query behaviors, and any suspicious activities that may indicate a security threat. For instance, it can detect uncommon occurrences like a surge in failed sign-in attempts using various credentials (indicative of a brute force attack), or the access of an SQL Server by a legitimate user from a compromised device that has previously communicated with a crypto-mining command-and-control (C&C) server.

[^8]: [Microsoft Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-cloud-introduction)

## Auditing
Database auditing is a critical component of data security, compliance and integrity. Auditing enables the detection of unauthorized actions, as well as the actions performed by authorized users. It reveals who did what, and what was affected. Many industries and regions have strict regulations that mandate auditing of database activities, for e.g. HIPAA, GDPR etc. 

The good news is that all Azure Managed Databases are equipped with comprehensive database auditing features. Once enabled, these audit logs can be seamlessly transmitted to Azure Log Analytics for short term storage and analytics. Beyond this, for organizations with long-term storage needs or compliance requirements, the audit logs can be exported to an Azure Storage account. 

## Compliance
To ensure the Azure Managed Databases adhere to the industry standards, regulations and organizational policies, you can make use of Azure Policy. Azure Policy provides a comprehensive library of pre-defined policy definitions based on industry best practices. These policies can be effortlessly applied to ensure proper configuration and security of your managed databases. 

With Azure Policy you can ensure that Azure Managed Databases comply to the Zero Trust architecture aspects described in this article. For example, disallowing the creation of Database with public endpoint, ensure that audit logging is configured for every managed database, disabling local or native database authentication and allowing only Microsoft Entra ID Authentication etc.     

## References
