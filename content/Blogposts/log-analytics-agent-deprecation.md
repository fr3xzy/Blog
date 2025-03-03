---
title: "Log Analytics Agent deprecation - Prepare for the end"
description: "On the 31st of August 2024, Microsoft will deprecate the Log Analytics VM Agent. This change has been announced years ago, but still many are not prepared. How can you assess your environment and migrate to the Azure Monitor Agent before the agent will stop working. Start cleaning up your environment now and you can relax all summer without stressing about your monitoring!"
date: "2024-03-06T18:27:58+01:00"
categories:
  - "Azure"
tags:
  - "Azure Automation"
  - "Azure Compute"
  - "KQL"
  - "Azure Spring Clean"
#menu: main # Optional, add page to a menu. Options: main, side, footer
draft: false
type: "blog"
# Theme-Defined params
thumbnail: "img/springclean24thumbnail.jpg" # Thumbnail image
lead: "Blogpost about the deprecation of the Log Analytics VM agent for Azure Spring Clean 2024" # Lead text
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

On the 31st of August 2024, Microsoft will deprecate the Log Analytics VM Agent. This change has been announced years ago, but still many are not prepared. How can you assess your environment and migrate to the Azure Monitor Agent before the agent will stop working. In this 2 part blog service, we will look at how you can find and migrate away from the Log Analytics Agent before the VM extension gets deprecated. In part 1 we will start by finding the affected VMs we will need to migrate.

Before we start, I would like to thank Thomas Thornton and Joe Carlyle that are hosting the Azure Spring Clean! And don't forget to check out the other amazing contributions to the community event over at the [Azure Spring clean website.](https://azurespringclean.com/)

## Affected Services
Even though it's the Log Analytics agent that is deprecating, the impact is larger than just monitoring. There are several services that are dependent on the same agent. Here is a list of some of the services that are affected by this deprecation:
- Sentinel
- Azure Automation Update Management
- Change Tracking/Inventory
- VM/AVD/Stack HCI Insights
- Defender for Cloud
- Network Watcher
- Container Monitoring Solution
- Azure Automation Agent-based Hybrid Worker 

A full list of migration recommendations for the services can be found [here.](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/azure-monitor-agent-migration#migrate-additional-services-and-features)

## OMS? MMA? AMA? WTF?
Before we dive into finding the agents, we need to establish the possible versions of VM extension you can find. You will see 2, maybe all 3 different abbreviations for VM extensions that is used for monitoring. First you have the OMS Agent, or the "Operations Management Suite Agent for Linux" as it's actually called. This is the legacy Log Analytics Agent for Linux VMs and this is one of the VM extensions that are deprecating. The other deprecating extension is the MMA, or Microsoft Monitoring Agent. This is the Log Analytics Agent for Windows. Lastly, we have the AMA. The AMA, or Azure Monitor Agent, is the new extension for both Linux and Windows VMs. Shortly summarized, OMS and MMA bad, AMA good!

## Finding affected VMs
So this can be done in a few different ways. I'm going to show you 2 different ways with different use-cases. The first way is to find VMs with the agent installed that is sending data to a Log Analytics Workspace. This would require you to know which Log Analytics Workspace the VMs are connected to, and sometimes there can be suprises. The second way will show you all VMs that have the Agent installed. With these 2 ways combined, you will know where you should look closer to prevent any services breaking on the 1st of September.

### Log Analytics query
To start, we will use a simple Log Analytics KQL query to find all VMs sending data to a specific workspace using the legacy agent. The query is as simple as this:
```SQL
Heartbeat
| where Category == "Direct Agent"
| summarize by Computer
```
And the result will be just as simple:
![Query results](/img/law-agent-query.png)
The query looks for logs in the Heartbeat table that have the Category *Direct Agent*. The heartbeat table is a log table that is used by the agent to give it's health status. We then use the *summarize* function to give us a list of only the names of the VMs sending logs to the workspace.

### Defender for Cloud and Resource Graph Queries
So this might sound a bit weird, but we can actually find the agents installed in Defender for Cloud. Let me explain, since Defender for Cloud use the Log Analytics Agent, it will give you the recommendation to install it, and it will show you which VMs have the agent installed and which doesn't. We can use this to our advantage. Either you can go into the Defender for Cloud view in the Azure portal and look through the recommendations, but the GUI can be a bit annoying with a lot of scrolling and clicks to get where you need to so you see all VMs with the agent installed. Instead, you can go into the Resource Graph Explorer and paste this query to get the same results:

```SQL
securityresources
    | where type == "microsoft.security/assessments"
    | extend source = trim(' ', tolower(tostring(properties.resourceDetails.Source)))
        | extend resourceId = trim(' ', tolower(tostring(case(
            source =~ "azure", properties.resourceDetails.Id,
            source =~ "aws" and isnotempty(tostring(properties.resourceDetails.ConnectorId)), properties.resourceDetails.Id,
            source =~ "gcp" and isnotempty(tostring(properties.resourceDetails.ConnectorId)), properties.resourceDetails.Id,
            source =~ 'aws', properties.resourceDetails.AzureResourceId,
            source =~ 'gcp', properties.resourceDetails.AzureResourceId,
            extract('^(.+)/providers/Microsoft.Security/assessments/.+$',1,id)
            ))))
    | extend status = trim(" ", tostring(properties.status.code))
    | extend cause = trim(" ", tostring(properties.status.cause))
    | extend assessmentKey = tostring(name)
    | where assessmentKey == "d1db3318-01ff-16de-29eb-28b344515626" and status == "Healthy"
    | extend vmName=split(resourceId,"/")[-1]
    | project vmName,resourceGroup,subscriptionId
```
This will give us this nice little list of which VMs have the agent installed:
![Resource Graph Explorer Results](/img/la-agent-query.png)

## Wrap-up
With these two queries, you can find both which VMs are connected to a specific workspace with the legacy agent and all VMs that have the extension installed. In the upcoming part 2 of this blogpost, we will look at what you need to do to migrate over to the Azure Monitor Agent. Thanks again to Thomas and Joe for hosting the Azure Spring Clean this year as well!