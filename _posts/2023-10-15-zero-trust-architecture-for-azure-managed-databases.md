---
title: "Zero Trust Architecture for Azure Managed Databases"
date: 2023-10-15
excerpt: "In this blog post I outline various aspects of building a Zero Trust architecture for Azure Managed Databases."
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
In this blog post I outline various aspects of building a Zero Trust architecture for Azure Managed Databases.

Zero Trust model states that you should never assume trust but instead continually validate trust. When users, applications, devices and data all resided inside the organization's firewall, they were assumed to be trusted. This assumed trust allowed for easy lateral movement after a malicious actor compromised an endpoint device.

Today, most users access applications and data from the internet, and many companies now allow users to use their own devices at work. Most components of a transactions, *the users, network, and devices*, are no longer completely under organizational control. The Zero Trust model assumes breach and verifies each request as though it originates from an open network. Regardless of where the request originates or what resource it accesses, Zero Trust teaches us to *"never trust, always verify."* Every access request is fully authenticated, authorized, and encrypted before granting access. No longer is trust assumed based on the location inside an organization's perimeter.

Zero Trust architecture is based on three principals:
1. Verify explicitly: Always authenticate and authorize based on all available data points, including user identity, location, device health, service or workload, data classification, and anomalies.
2. Use least-privilege access: Limit user access with just-in-time and just-enough access (JIT/JEA), risk-based adaptive polices, and data protection to help secure both data and productivity.
3. Assume breach: Minimize blast radius and segment access. Verify end-to-end encryption and use analytics to get visibility, drive threat detection, and improve defenses.

## Architecture
<img alt="Zero Trust Architecture diagram for Azure Managed Databases" src="/assets/images/zero-trust-arch/zero-trust-arch-for-azure-databases.svg">

## Network Isolation
Azure managed databases offer multiple options like Private Endpoints, VNET Integration, Firewall rules, disable internet access etc. when it comes to implementing network isolation. The idea is to **assume breach** and minimize blast radius by blocking any direct internet access and allowing traffic from only designated VNETs/IP ranges. 

In the above architecture diagram, Private Endpoint[^1] feature is used to bring the database into the specific subnet inside a Virtual Network (VNET). Direct public network access is disabled. Additionally, a Network Security Group (NSG) is configured at the subnet level which acts as a firewall that allows you to control inbound and outbound network traffic for that subnet.  

[^1]: [Private Endpoint](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-overview)

## Access Control
One of the most important principals of Zero Trust architecture is Access Control. Integrate Azure Managed database with Microsoft Entra ID and disable native database username and password access. Any user or workload must connect with the database using their own identity assigned via Microsoft Entra ID. This allows you to be free from storing database credentials in config files, Key vault, Kubernetes secrets etc. and instantly reduces the security risks related with database password leakage. 

### User Access
Define security groups in Microsoft Entra ID for different personas which are expected to access the database, for e.g., Database Administrators, Database Developers, Database Support etc. Each of these groups are mapped to appropriate roles like admin, writer, reader etc. inside the database following the principal of **least-privilege access**. 

Users should not be given continuous 24/7 access to the database. Any user who requires access must go through an approval process which can be achieved by enrolling the security groups 

Enable Privileged Identity Management (PIM) for the security groups defined earlier. This allows you to grant users just-in-time membership to these groups. Any time a user needs to access the database, they must raise a request through PIM which goes through an approval process. Once approved, the user is granted temporary time-bound access to the designated Microsoft Entra ID group which is further mapped to appropriate role in the database. Now the user can log into the database using their own identity and perform their tasks.

Since the groups are enrolled under PIM, you can define polices like time-bound access, require justification, approval process, multi-factor authentication, conditional access etc.   

<p align="left">
  <img alt="Privileged Identity Management options screenshot" src="/assets/images/zero-trust-arch/pim-settings.jpg" style="max-width: 350px;border: 1px solid black;"/>
</p>

### Workload Access 
To enable workloads (such as an application, service, script, or container) to interact with Azure Managed databases, Azure provides the feature of *Workload Identity*. A workload identity is an identity assigned to the software workload to authenticate and access other services and resources. Azure supports three types of workload identities - *Service Principals*, *Managed Identity* and *Application*[^1]. It is recommended to use Managed Identity where possible instead of a Service Principal. With Service Principal you are responsible for managing and rotating secrets, whereas with Managed Identity, Microsoft Entra ID automatically manages and rotates the secrets for you. Additionally, the secrets are not exposed by Managed Identity, the application can make use of Azure.Identity Library[^2] to consume the secrets at runtime.  

[^1]: [Microsoft Entra ID - Workload Identity](https://learn.microsoft.com/en-us/azure/active-directory/workload-identities/workload-identities-overview)
[^2]: [Azure Identity Client Libraries](https://learn.microsoft.com/en-us/azure/aks/workload-identity-overview?tabs=dotnet#azure-identity-client-libraries)

Following is an example architecture where an application running on Azure Kubernetes Service (AKS) makes use of Workload Identity to authenticate and connect with Azure MySQL database:  

<img alt="Connect with Azure MySQL Database using Azure Kubernetes Service (AKS) Workload Identity architecture diagram" src="/assets/images/zero-trust-arch/aks-workload-identity.svg">


In this example, Azure MySQL database is integrated with Microsoft Entra ID and an Azure Managed Identity is setup to 
- The OIDC feature lets the AKS cluster act as an token issuer, issuing tokens to the Kubernetes Service Accounts. 
- The Workload Identity feature deploys a mutating webhook admissions controller which injects identity details in the pod.
Further, the Azure Managed Identity is configured to trust the access tokens issued by AKS. At runtime, the workload can exchange the service account token projected to its volume for an Entra ID access token via the use of Azure Identity Library and then use that token to connect with the database. 

### DevOps Access

## Data Protection
- Azure Managed Databases support data encryption at rest by using Azure managed key or customer-managed key. With customer-managed key, you fully control data access by the ability to remove the key and make the database inaccessible. You have full control over they key lifecycle, including rotation of key to align with corporate policies. This also allows you to implement separation of duties for managing keys and data. 
- Azure Managed Databases use SSL/TLS to encrypt the data in transit. 

## Auditing
Database auditing is a critical component of data security, compliance and integrity. Auditing enables the detection of unauthorized actions, as well as the actions performed by authorized users. It reveals who did what, and what was affected. Many industries and regions have strict regulations that mandate auditing of database activities, for e.g. HIPAA, GDPR etc. 

All Azure Managed Databases support database auditing features. Once turned on, the audit logs can be send to Azure Log Analytics. For long term storage requirements the audit logs can be exported to Azure Storage account. 

## References
