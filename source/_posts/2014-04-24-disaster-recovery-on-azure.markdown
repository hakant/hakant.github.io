---
layout: post
title: "Disaster Recovery on Azure"
date: 2014-04-24 19:30:37 +0200
comments: true
categories: [azure, architecture]
footer: true
sharing: true
---

There are things we tend to ignore. Things that we don't want to spend time on or think about because there are no immediate benefits. Disaster recovery is one of those topics. The chances are fairly low, but the impact on our business is catastrophic. The consequences can go as far as losing the business completely. Unless there is some sort of a disaster recovery plan in place.

Because I'm working with Microsoft Azure on a daily basis as part of my job, I'll be focusing on Azure but the fundamental principles and concepts should be universal for every cloud platform, or even for on premises type scenarios.

## Two key objectives
<br>
>##### Recovery Time Objective (RTO)
Recovery Time Objective is the maximum amount of time allocated for restoring application functionality.

This is usually a requirement coming from the business. Basically the question is how critical your platform is and how much down time you can tolerate in such a catastrophic event.

>##### Recovery Point Objective (RPO)
The recovery point objective (RPO) is the acceptable time window of lost data due to the recovery process. 

For example, if the RPO is one hour, you must completely back up or replicate the data at least every hour. Once you bring up the application in an alternate datacenter, the backup data may be missing up to an hour of data. Like RTO, critical applications target a much smaller RPO.

These two key objectives determine the approach that needs to be followed and therefore the effort and cost of the whole disaster recovery plan.

The purpose of this blog post is not to describe solutions for each possible combination of these two objectives, but to describe a couple of approaches that can tolerate relatively long recovery time objective (~24 hours) with a more aggressive (so short) recovery point objective.

Don't forget that having a short RTO and RPO together can be both costly and complex to implement depending on the storage needs of your applications.

## Running business on a single datacenter

Let's say you haven't thought on a disaster recovery plan yet. If your software is deployed on a single datacenter and making use of Azure Storage and/or Azure SQL Database you are at risk of losing your business should your datacenter goes dark due to a disaster or a catastrophic failure.

An Azure datacenter is equipped with fault domains and redundancy to keep your service highly available but these are all inside the datacenter. If the whole datacenter goes down all the compute instances, databases and storage services will go down with it.

![Single Region Deployment](/assets/Disaster_Recovery_On_Azure/Single_Region_Deployment.png)

Assuming that you still have your software in-house somewhere, there should be no risk of losing the compute instances forever. By creating and deploying the cloud packages on another datacenter the compute instances can be recovered. __The key to business continuity is to be able to recover the data that is stored on Azure Storage and Azure SQL Databases.__

## Frequently backup data outside the datacenter

Whatever the recovery objectives are, backing up application data outside of the datacenter is a must for business continuity. How frequently the data is backed up or synced outside the datacenter will determine the recovery point objective.

When there are backups available outside the datacenter, the environment can be moved to another datacenter by restoring the data and installing compute instances using existing cloud service packages.

![Redeploy to another datacenter](/assets/Disaster_Recovery_On_Azure/Redeploy_Azure_Datacenter.png)

This is called "redeploy" recovery model and as you might already guess has a long recovery time objective. All the individual pieces of the environment needs to be moved to another datacenter and redeployed. 

## How to backup Azure data (together with RPO and RTO considerations) 
<br><br>
#### 1. Azure Storage
The good news is that Azure Storage Service has built-in replication strategies. Two of them are geographical redundancy, meaning that all storage data is replicated across datacenters. That's exactly what disaster recovery is about.

![Azure Storage Geo Redundancy](/assets/Disaster_Recovery_On_Azure/Azure_Storage_Redundancy.png)

The difference between "Geo Redundant" and "Read-Access Geo Redundant" is that the latter allows the redundant data to be accessed at all times in a read-only fashion. This brings us to another important point:

>Here is what Microsoft's documentation say about the estimated azure storage failover time in case of a disaster:  "estimated time that the data will be accessible to customers after a disaster is 24 hours."

In the light of these information 3 options appear for backing up the Azure Storage:

* __Geo-Redundancy__: There is no SLA but RPO is practically very short. RTO is long. Only after about 24 hours the failover process will be completed by Microsoft and the data will be available again.
* __Read-Access Geo Redundant__: RPO is again the same as above. However almost immediately the redundant data can be accessed (read-only) which allows systems to run in a degraded mode (if they're designed in such a fault tolerant way of course). Then again in 24 hours everything will be fully operational.
* __Custom__: If RTO needs to be shorter, the only option is to look for a third party product that can replicate an azure storage to another datacenter. This redundant storage will always be available in Read/Write mode (since it's just another regular storage account, only in another geographical region).

<br>
#### 2. Azure SQL Databases
<br>







