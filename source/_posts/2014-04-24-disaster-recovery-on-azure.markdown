---
layout: post
title: "Disaster Recovery on Azure"
date: 2014-04-27 19:30:37 +0200
comments: true
categories: [azure, architecture]
footer: true
sharing: true
---
<br/><br/>

>####UPDATE: 
Things keep changing & improving on Azure. This post may not reflect the latest capabilities in terms of Disaster Recovery for Azure Applications. Make sure to read the subject on [MSDN](https://msdn.microsoft.com/en-us/library/azure/dn251004.aspx) as well.

There are things we tend to ignore. Things that we don't want to spend time on or think about because there are no immediate benefits. Disaster recovery is one of those topics. The chances are fairly low, but the impact on our business is catastrophic. The consequences can go as far as losing the business completely. Unless there is some sort of a disaster recovery plan in place.

Because I'm working with Microsoft Azure on a daily basis as part of my job, I'll be focusing on Azure but the fundamental principles and concepts should be universal for every cloud platform, or even for on premises type scenarios.

## Two key objectives
<br>
>##### Recovery Time Objective (RTO)
Recovery Time Objective is the maximum amount of time allocated for restoring application functionality.

This is usually a requirement coming from the business. Basically the question is how critical is your platform and how much down time can you tolerate in such a catastrophic event?

>##### Recovery Point Objective (RPO)
The recovery point objective (RPO) is the acceptable time window of lost data due to the recovery process. 

For example, if the RPO is one hour, you must completely back up or replicate the data at least every hour. Once you bring up the application in an alternate datacenter, the backup data may be missing up to an hour of data. Like RTO, critical applications target a much smaller RPO.

These two key objectives determine the approach that needs to be followed and therefore the effort and cost of the whole disaster recovery plan.

The purpose of this blog post is not to describe solutions for each possible combination of these two objectives, but to describe a couple of approaches that can tolerate relatively long recovery time objective (~24 hours) with a more aggressive (so short) recovery point objective.

Don't forget that having a short RTO and RPO together can be both costly and complex to implement depending on the storage needs of your applications.

## Running business on a single datacenter

Let's say you haven't thought on a disaster recovery plan yet. If your software is deployed on a single datacenter and making use of Azure Storage and/or Azure SQL Database you are at risk of losing your business should your datacenter goes dark due to a disaster or a catastrophic failure.

An Azure datacenter is equipped with fault domains and redundancy to keep your service highly available but these are all inside the datacenter. If the whole datacenter goes down all the compute instances, databases and storage services will go down with it.

{% img /assets/Disaster_Recovery_On_Azure/Single_Region_Deployment.png 500 438 'Single Region Deployment' %}

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

Yesterday I woke up to an [annoucement from Microsoft](http://view.email.microsoftemail.com/?j=fe9016787161057a71&m=fe621570756503797d1c&ls=fe1817787c60027e711d79&l=fec21c767365017e&s=fe2212717465037a701d78&jb=ff5e177873&ju=fe5711777060017b7611) introducing new types of databases in Azure with new disaster recovery features:

![Disaster Recovery Per Database Type](/assets/Disaster_Recovery_On_Azure/DisasterRecoveryPerDatabaseType.png)

Let's look at each option and see what it means:

* __Restore to an alternate	Azure region__: This phrase means "no silver bullet". Basic type database owners are responsible with their own backup and restore operations. Luckily there is a convenient way to backup and restore an Azure SQL Database regardless of its type.
<br>

##### Automatic Database Export:

Actually there are a couple of ways to replicate or backup a database. But most of these options have their own shortcomings for disaster recovery. There is a feature called [Database Copy](http://msdn.microsoft.com/en-US/library/azure/ff951624.aspx) which creates a [transactionally consistent replica](http://technet.microsoft.com/en-us/library/ms151176.aspx) of the source database in __the same datacenter.__ Because the replica resides in the same datacenter there is no geo-redundancy. But after the copying is completed, this database can be exported to a storage account in another datacenter. 

This is exactly what __Automatic Database Export__ feature does. It first replicates the database with a copy operation, thus getting a transactionally consistent copy of the database, then exports it to the storage account that is configured. To see how it is configured you can visit [this blog post](http://blogs.msdn.com/b/sql-bi-sap-cloud-crm_all_in_one_place/archive/2013/07/24/sql-azure-automated-database-export.aspx).

A direct manual export operation itself does not generate a transactionally consistent copy of a database. This means you may end up having an Order item in the database with a missing OrderDetails. Automatic Database Export however takes care of this problem. The documentation was not very clear on that so I asked the man himself:
<br>

<blockquote class="twitter-tweet" lang="en"><p><a href="https://twitter.com/hakant">@hakant</a> <a href="https://twitter.com/Azure">@Azure</a> yes I&#39;m pretty sure it is</p>&mdash; Scott Guthrie (@scottgu) <a href="https://twitter.com/scottgu/statuses/460180886915276801">April 26, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

So we're on the right track here.
<br>

>__Attention:__ First part of Automatic Database Export is the replication of the database with Database COPY operation which runs two databases at the same time. This means export operations are partially doubling the database costs.

So far so good... But what determines RTO and RPO for Automatic Database Export? This analysis depends on the frequency and the destination storage configured for the database export operation.
<br>

- _Exporting to a separate storage account on another datacenter:_

In this scenario, the RPO will simply be around [Database Export Frequency]. If the database is exported every 6 hours, in the worst case scenario 6 hours of data may be lost. 

RTO is simply the [Database Restore Duration]. No need to wait for Microsoft to execute the Azure Storage failover process. The database export file is immediately available on the storage account in the other datacenter.
<br>

- _Exporting to the primary storage account on the same datacenter:_

In this scenario, we're relying on the Automated Azure Storage Geo-Replication to get the exported database transferred to another datacenter. Since this scenario uses the geo-replication feature of the storage account there won't be any additional costs for geo-reduntant storage. This is nice indeed.

But in the worst case scenario the RPO can go as far as [2 * Database Export Frequency] if the datacenter happens to fail during the geo-replication process. In that case, the last export file won't be available after the failure but only the one before that will be. 

On the other hand, the RTO is [Estimated Azure Storage Geo-Failover Time + Database Restore Duration]. Again, Microsoft's estimation for Azure Storage geo-failover is around 24 hours.

So the moral of the story is that having a dedicated separate Azure Storage account on the other datacenter is beneficial for both RPO and RTO. But as usual the downside is financial. The separate storage account is also billed separately. Both for storage and bandwith to transfer the data. Though it's also worth mentioning that Azure Storage is [fairly cheap](http://azure.microsoft.com/en-us/pricing/details/storage/).

Now let's move to the other disaster recovery features.

* __Geo-Replication Passive Replica__: Unfortunately there is no documentation available for this mysterious feature yet. Frankly there is no single mention of this feature anywhere else at the time of this writing. So the technique described above for the Basic databases also applies here until Microsoft actually ships this feature or shares more about it. I'll update this section when that happens.

* __Active Geo-Replication__: This one is a silver bullet solution. Unfortunately only the Premium database owners can make use of this feature. With Active Geo-Replication, you can create and maintain up to four readable secondary databases across geographic regions. Basically this gives a very short RPO and RTO for the price of running multiple Premium databases ([which is quite a lot](http://azure.microsoft.com/en-us/pricing/details/sql-database/#basic-standard-and-premium)). You can read more about [Active Geo-Replication on MSDN](http://msdn.microsoft.com/en-US/library/azure/dn741339.aspx).

## Final thoughts

If your business can tolerate larger than ~24 hrs recovery time objective (RTO) Azure provides a couple of inexpensive features that you can already start making use of. These features also require very little effort to setup and you guarantee business continuity in case of a catastrophic event.

The most cost effective strategy is the following:

1. Make sure your storage account is configured for Geo-Redundancy (default).
2. Turn on Automatic Database Export and configure a high frequency that makes sense. Keep track of how long this operation takes and adjust your frequency based on that as well.
3. Set your default azure storage account as the destination of automatic database export. So rely on the geo-redundancy of your storage account.

If you need a shorter RTO build up on this strategy. This post is mainly focused on "Redeploy" pattern. There are other patterns available on MSDN and some are focused on more aggressive RTO's. I recommend reading: [Disaster Recovery and High Availability for Azure Applications](http://msdn.microsoft.com/en-us/library/dn251004.aspx).

## References

Most information on this blog post is obtained from the following MSDN pages. Some others are direct observations & usage from Azure Management Portal.

* [Disaster Recovery and High Availability for Azure Applications](http://msdn.microsoft.com/en-us/library/dn251004.aspx)
* [Azure SQL Database Business Continuity](http://msdn.microsoft.com/library/azure/hh852669.aspx)
* [Copying Databases in Azure SQL Database](http://msdn.microsoft.com/en-US/library/azure/ff951624.aspx)
* [How to: Import and Export a Database (Azure SQL Database)](http://msdn.microsoft.com/en-US/library/azure/hh335292.aspx)
* [SQL Database updates coming soon to the Premium preview](http://blogs.msdn.com/b/windowsazure/archive/2014/04/04/sql-database-updates-coming-soon-to-the-premium-preview.aspx)









