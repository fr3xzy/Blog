---
title: "VM operations in Azure without RDP"
description: ""
date: "2022-01-11T10:36:34+01:00"
categories:
  - "Azure"
tags:
  - "Azure Compute"
  - "Azure VM"
  - "Operations"
  - "Azure Automation"
#menu: main # Optional, add page to a menu. Options: main, side, footer
draft: false
type: "blog"
# Theme-Defined params
thumbnail: "img/azure-operations-wo-rdp.PNG" # Thumbnail image
lead: "How do you manage your VMs in Azure without needing to RDP to them?" # Lead text
comments: false # Enable Disqus comments for specific page
authorbox: true # Enable authorbox for specific page
pager: true # Enable pager navigation (prev/next) for specific page
toc: false # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
sidebar: "right" # Enable sidebar (on the right side) per page
widgets: # Enable sidebar widgets in given order per page
  - "search"
  - "recent"
  - "taglist"
  - "social"
---

The "Traditional" way to manage servers was to RDP or SSH into them and apply the changes you needed to do. And this was regarded as safe to do (Not by everyone of course) since you usually already were inside the office or datacenter. But when your servers are in another datacenter or even on the other side of the world, what can you do then?
You can just open up RDP for your IP-address or a whole range of IP-addresses, but it is not recommended.

In the [2020 Unit 42 Incident Response and Data Breach Report](https://www.paloaltonetworks.com/resources/research/2020-unit42-incident-response-and-data-breach-report) published by Palo alto Networks, it is reported that 50% of initial attacks was through RDP.
There are services in Azure that makes using RDP safer (like Bastion or JIT-access), but in this series I will focus on services you can use to manage VMs without RDP.
This article series will contain this post as an introduction before going into the different Azure services.

Here are some services that can help you do common operations on your VM from the Azure Portal:

## Run scripts on your VM with Run command
Let's say you want to change a registry key or want to return a setting to the portal, the Run Command feature helps you! With Run Command you can choose from a set of pre-configured scripts from Microsoft, or you can run a custom script. Run command work for both Windows and Linux VMs, but there is more premade script for Windows.

There are some limitations to what you can do with Run Command custom scripts, here are a few of those restrictions:
- Output is limited to the last 4,096 bytes.
- You can't cancel a running script.
- Only one script can run at a time
- The maximum time a script can run is 90 minutes
- Script that prompt for input is not supported.

You can find the Run Command feature in the portal by navigating to your VM and scroll down the left menu till you find the Run Command blade:

![Run command feature](/Blog/img/Azure-run-command.PNG)

## Troubleshoot a network connection with Network watcher
In Azure there is a lot of networking. And in the cloud with software defined networking, it might not always be as easy as "check if the cable is connected". A feature that will help you with troubleshooting network connections in Azure is the Network Watcher, also known as "Connection Troubleshoot".

![Connection Troubleshoot](/Blog/img/Connection-troubleshoot.PNG)

With Network Watcher you specify which direction you want to test, inbound or outbound. Next you choose what's your desired destination or source. Here you can choose your current IP-address, a specified IP-address, or an [Azure service tag](https://docs.microsoft.com/en-us/azure/virtual-network/service-tags-overview#available-service-tags).

Finally, you select what port you want to test. There are several commonly used services and ports that are pre-configured. So don't worry if you don't remember what port PostgreSQL uses! If you don't find the service or port in the pre-configured list, you can also specify a custom port and what protocol it uses.

When you click **check connection** Network Watcher will check if the port and protocol specified is allowed in or out of the VM. Now, the Connection troubleshoot will only check if the traffic is allowed or blocked in the Network Security Group for the VM and correlating Subnet. That means that even if Connection Troubleshoot says the traffic is allowed, it might still not work. There is thankfully a detailed version of the connection troubleshooter that is linked on the connection troubleshoot page.

## Update management using Azure Automation
But what about Security patches and other updates? I don't want the VM to install them whenever they are released without me knowing. Well, there is a solution for that too, and its way less complicated than creating a SCCM server and configuring WSUS on all servers.

In Azure Automation there is a feature that is called Update Management. Update Management is a free feature of Azure Automation that helps you patch both your Windows and Linux servers. With Update Management you get a centralized page where you can see and configure:
- VMs to enable the feature on
- Check compliance of VMs
- Define deployment schedule
- Deployment status

## Deploy application to VMs with Azure compute galleries
In the initial config after deploying a VM, you might want to install an application on the VM for it to function. Microsoft changed the formerly known Shared Image Gallery to Azure compute gallery. The reason for the name change is that in addition to storing and sharing images, you can now do the same for application packages.

With Azure compute gallery you get the benefits of:
- Grouping and versioning of packages.
- Control access with Azure Role Based Access Control (RBAC).
- Install from storage account without a direct internet connection.
- Automatic deployment with a DeployIfNotExists policy.

For example, let's say that you want to update antivirus on all your servers. So instead of RDPing in to all your servers and update or install the new version, you can create an application package in Azure Compute Gallery and create an Azure Policy to automatically deploy the software to servers.

## Collect logs from your VM in the portal with Azure VM Inspector
A first step in troubleshooting an application or service is often checking the logs on the server. In Azure there is a feature for VMs to collect event logs, configurations, settings, and registry keys. Then read through the report directly from the portal. This feature is called VM Inspector and is per January of 2022 still in preview. That means that you will have to enable the feature for your subscription. 

**Note:** Microsoft does not recommend using preview features in production and the price of a service may change when it goes globally available.

There are some prerequisites for VM Inspector. You need to have managed disks on your VM, the disks can't be encrypted, and since it is in preview there are a limited amount of regions where it is available.
After you have enabled VM Inspector on your subscription you need to connect it to an existing storage account or creating a new one. Once you have connected it to a storage account, you can create your first report! After the report is finished, it will be stored as a zip-file in the connected storage account.

This tool combined with the Run Command feature makes it possible to troubleshoot and fix most configuration and other simple errors remotely!

## Manage certificates and secrets with Azure Key Vault
Management of certificates can be an unappreciated job. It can include waking up at 07:00 on a Sunday to change a certificate and getting yelled at Monday morning cause an undocumented application that used the certificate is now partly unavailable for users. All this headache could be avoided by using Azure Key Vault and some of its features for managing and distributing certificates and secrets!

Azure Key Vault lets you generate, import, or even buy a managed certificate. With generated and managed certificates you can configure automatic renewal of the certificates. You can then install the Azure Key Vault extension on your VMs and have them automatically deploy the new versions of the certificate to the VM.

## Conclusion
In this blog I gave a quick introduction to some tools and features in Azure that can help you manage your servers on an OS level without needing to RDP into them. Going forward I will create follow up blogs going further into these features.