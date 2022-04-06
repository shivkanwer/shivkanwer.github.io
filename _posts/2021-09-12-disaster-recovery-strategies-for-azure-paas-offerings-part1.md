---
title: "Disaster Recovery Strategies for Azure PaaS offerings - Part 1"
date: 2021-09-12
classes: wide
categories:
  - Architecture
tags:
  - azure service
  - disaster recovery
  - PaaS
---
Planning a Disaster Recovery (DR) Strategy for the Azure Infra based workloads is quite straight forward (that does not mean it is easy) with the use of services like Azure Site Recovery, Azure Backup etc. However, when it comes to putting up a DR strategy for a PaaS based solution the story is quite different especially when a solution involves multiple Azure PaaS Services. Every PaaS service has its own way of implementing DR:
* Some PaaS services provide out of the box features for DR
* Some PaaS services support auto-failover whereas in others the failover needs to be initiated by the customer.
* In some PaaS services there is a clear path for doing a failback whereas in some other services customer needs to implement extra controls to do a failback.
* For certain PaaS services customers need to change the client configuration to connect with the secondary instance.
So I have decided to put together a series of articles for covering the DR strategies for various Azure PaaS offerings.

### How should one decide when to failover?
Failover should be based on the RPO (Recovery Point Objective) and RTO (Recovery Time Objective) that has been decided for your workload. Do not wait for Azure to act. For e.g. If the recovery time object for your application is 30 mins and for Azure to do a failover it requires 1 hours, then you should act and initiate the failover from your end (Use automation and health monitoring).

One of the first things you should check is the [paired Azure region](https://docs.microsoft.com/en-us/azure/best-practices-availability-paired-regions#azure-regional-pairs) for your primary region and make sure if the service that you are looking to DR is [available in the paired region](https://azure.microsoft.com/en-in/global-infrastructure/services/). Not all Azure services are available in every Azure region. If the service is not available in the paired region look for next closest region.
{: .notice--primary}

### Services covered in part 1
* Azure Container Registry (ACR)
* Azure Kubernetes Service (AKS)
* Azure SQL Managed Instances
* Azure SQL Databases
* Azure Service Bus
* Azure Key Vault
* Azure DevOps

## Azure Container Registry (ACR)