---
title: "Reduce your Azure cost by finding unused resources"
description: ""
date: "2023-03-15T10:00:00+01:00"
categories:
  - "Azure"
tags:
  - "KQL"
  - "Cost Management"
  - "Azure Spring Clean"
  - "Azure Resource graph"
#menu: main # Optional, add page to a menu. Options: main, side, footer
draft: false
type: "blog"
# Theme-Defined params
thumbnail: "img/springcleanthumbnail.png" # Thumbnail image
lead: "As a contribution for Azure Spring Clean, I wrote a blogpost about how to use KQL to find orphaned resources to help reduce your cloud waste." # Lead text
comments: false # Enable Disqus comments for specific page
authorbox: false # Enable authorbox for specific page
pager: false # Enable pager navigation (prev/next) for specific page
toc: false # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
sidebar: "right" # Enable sidebar (on the right side) per page
widgets: # Enable sidebar widgets in given order per page
  - "recent"
  - "taglist"
  - "social"
---

In Microsoft's Well-Architected Framework there is a pillar for Cost Optimization. Some of the principals for optimizing cost are to continuously look for and clean up orphaned resources like disks and public IPs. In this blog I will show you some KQL queries that will help you find these resources and considerations you should take before cleaning them up. You can then use these queries in an Azure Workbook or dashboard to continuously review your environment. Before I start, I would like to thank [Thomas Thornton](https://twitter.com/tamstar1234) and [Joe Carlyle](https://twitter.com/wedoAzure) for organizing the [Azure Spring Clean](https://www.azurespringclean.com/). Please go check out the awesome content that other people in the community have created for the event!


## Azure Well-Architected Framework

[The Azure Well-Architected Framework](https://learn.microsoft.com/en-us/azure/architecture/framework/) is a framework from Microsoft that gives guidance on some key pillars. These pillars are [Reliability](https://learn.microsoft.com/en-us/azure/architecture/framework/resiliency/overview), [Security](https://learn.microsoft.com/en-us/azure/architecture/framework/security/overview), [Cost](https://learn.microsoft.com/en-us/azure/architecture/framework/cost/overview), [Operational Excellence](https://learn.microsoft.com/en-us/azure/architecture/framework/devops/overview), and [Performance Efficiency](https://learn.microsoft.com/en-us/azure/architecture/framework/scalability/overview). These are also the pillars that you can find recommendations on in Azure Advisor. The framework will give you guidance, checklists, and recommendations on best practices when designing, managing, monitoring, and optimizing Azure Resources.

If you have ever done an [Azure Well-Architected Review](https://learn.microsoft.com/en-us/assessments/azure-architecture-review/) on the Cost pillar, which is an assessment/questionnaire from Microsoft that gives you recommendations on improvements on a specific pillar from WAF, you might have seen a recommendation called ["Look for Public IPs and orphaned NICs"](https://learn.microsoft.com/en-us/azure/architecture/framework/services/networking/ip-addresses/cost-optimization#configuration-recommendations). This recommendation is action is categorized by Microsoft as Critical. And in my experience with helping customers reduce their Azure cost, cleaning up orphaned resources can give a significant cut in cost. In addition to Public IPs and NICs, I will show you some KQL queries that will help you find orphaned disks and load balancers without backend pools.


## How do resources become *orphaned*?

An orphaned resource is a resource that no longer is connected to another required resource. An example could be a disk that is not connected to a VM or a Public IP not connected to a NIC. These resources are very likely to not being used, but like many Azure resources, they can still have a cost. By cleaning up orphaned resources, you could optimize your Azure cost. Remember, it's not "pay for what you use", it's "Pay-As-You-Go".

But how do these resources become orphaned? If you started working with Azure before the summer of 2022, you might know that it used to be that Azure did not ask if you wanted to delete the connected disks, public IPs and NICs when deleting a VM. This meant that you would need to manually delete all the resources connected to a VM, and this can be error prone. Because of this, a lot of customers are not aware that they have orphaned resources. Let's have a look on how to find them.


## Public IPs and disks

We will start with some simple ones, Public IPs and Disks. To find these resources, we will use the Azure Resource Graph Explorer to create some KQL queries. All the queries will be to the Resources table and all the result will be formatted to get links and friendly names for resource groups and subscriptions.
For Public IPs we will look for public IPs where the property "ipConfigurations" is empty. This will usually indicate that the resource is orphaned. An exemption for this is if the IP is used by a NAT gateway, so remember to double check resources before deleting them! The query for Public IPs looks like this:

```SQL
resources
| where type == "microsoft.network/publicipaddresses"
| where properties['ipConfiguration'] == ""
| project name,location,resourceGroup,subscriptionId,sku.name
```

And will give a result like this:
![orphaned public ips query result](/img/orphaned-pips.PNG)

So, we select the Name, Location, Resource group, Subscription, and SKU name. We chose to show the SKU of the public IP because it is only Standard SKU public IP that have a cost when it's not connected to a resource. Basic SKU IPs will not generate a cost when orphaned.

On to disks! Here we are looking for disks where the property "managedBy" is empty. The query looks like this:

```SQL
resources
| where type == "microsoft.compute/disks"
| where managedBy == ""
| project name,location,resourceGroup,subscriptionId,sku.tier,properties.diskSizeGB
```

And the results look like this:
![orphaned disks query result](/img/orphaned-disks.PNG)

Here, in addition to the identifying properties, we also show the SKU tier and the Size of the disk. This will help us to see right away which of the disks that have the biggest cost. When looking at orphaned disks, you should try to identify why the disk is left over. Is it by mistake or did someone intend to keep the disk as a backup of a deleted VM? For the latter, you should move the disk to another and cheaper storage solution like cold or archive storage in a Storage Account.

## Network Interfaces

This one is a bit more complex, but still simple. Now, we could do a simple query and return every NIC that is not connected to a VM, but it's not the NICs that we are interested in. We are interested in NICs that are not connected to a VM **and** has a public IP. Cause it is that potential Public IP that is costing us money. Let's have a look at the query and explain it before we look at the results:

```SQL
resources
| where type == "microsoft.network/networkinterfaces"
| where properties.virtualMachine == ""
| extend p=parse_json(properties.ipConfigurations.[0].properties)
| where p.publicIPAddress != ""
| extend publicIpSku=p.publicIPAddress.sku.name
| project name,location,resourceGroup,subscriptionId,publicIpSku
```

To only get NICs that have a connected public IP we need to dig into the properties. For that we use the [Extend operator](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/extendoperator) together with the [parse_json function](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/parsejsonfunction). This makes the query a bit more reader-friendly and keeps us from writing *properties.ipConfigurations.[0].properties.publicIPAddress* several times over. And we also use it to get a better name for the property of public IP SKU that we want in the results. Speaking of the results, let's have a look at what this query gives us.

![orphaned nics query results](/img/orphaned-nics.PNG)

Same as with Public IPs, only Standard SKU Public IPs will create a cost.

## Load Balancers

With load balancers, there are 2 things that can reduce cost. Both the load balancer itself and if the load balancer has a public IP associated. And since the load balancer is always running, we also pay for Basic SKU Public IPs.

```SQL
resources
| where type == "microsoft.network/loadbalancers"
| extend pipId=parse_json(properties.frontendIPConfigurations[0].properties.publicIPAddress.id)
| where properties.backendAddressPools == "[]"
| project name,location,resourceGroup,subscriptionId,sku.name,pipId
```

Here again, we use the extend operator to make the property of Public IP more reader friendly. And we also get the SKU of the load balancer itself. Both will help give us a potential cost reduction. For the finale time, here is the results of the query:

![orphaned lbs query results](/img/orphaned-lbs.PNG)

As you can see, the column for pipId isn't user-friendly, but it shows us if the load balancer has a connected Public IP or not. If the load balancer does not have a connected Public IP, the column will be empty.

## Creating a dashboard with the queries

So now that you have the queries, you probably want a central place to see them and monitor them. To achieves this, we can add them to an Azure dashboard. To do this, you can press the button for "Pin to dashboard" that is located right above the results pane.

![Pin to Dashboard](/img/orphaned-pintodash.png)

Looking at the dashboard after adding all the queries, it looks like this:

![Dashboard of queries](/img/orphaned-dashboard.PNG)

You could also add these queries to an Azure Workbook and add other queries and graphs to create a more complete Cost management monitoring collection.