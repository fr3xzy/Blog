---
title: "Introduction to Automanage"
description: ""
date: "2022-04-06T13:53:13+02:00"
categories:
  - "Azure"
tags:
  - "Governance"
  - "Compute"
  - "Azure VMs"
#menu: main # Optional, add page to a menu. Options: main, side, footer
draft: true
type: "blog"
# Theme-Defined params
thumbnail: "img/placeholder.png" # Thumbnail image
lead: "Get a intro to Azure Automanage and set-up best practice configuration for you new and existing VMs." # Lead text
comments: false # Enable Disqus comments for specific page
authorbox: true # Enable authorbox for specific page
pager: false # Enable pager navigation (prev/next) for specific page
toc: false # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
sidebar: "right" # Enable sidebar (on the right side) per page
widgets: # Enable sidebar widgets in given order per page
  - "recent"
  - "taglist"
  - "social"
---

What if you could make configuring the supporting services of your VMs so much easier? And without needing to assign and manage several policies. This is where Azure Automanage comes flying in like the saviour you always needed!

## What is Automanage?

Automanage is as simple as a configuration profile that you apply to your VMs that will automatically configure the services that are considered to be best practice for VMs in Azure. For example Azure Log Analytics and Azure Backup.

Microsoft even created some predefined automanage profiles, one for Production workloads and one for Dev/Test. So which services are configurable in Automanage?

| Service                       | Configuration Profile | Supported Scenario        |
|-------------------------------|-----------------------|---------------------------|
| Machines Insights Monitoring  | Production            | Windows, Linux, Azure Arc |
| Backup                        | Production            | Windows, Linux            |
| Microsoft Defender for Cloud  | Production, Dev/Test  | Windows, Linux            |
| Microsoft Antimalware         | Production, Dev/Test  | Windows                   |
| Update Management             | Production, Dev/Test  | Windows, Linux, Azure Arc |
| Change Tracking and Inventory | Production, Dev/Test  | Windows, Linux, Azure Arc |
| Guest configuration           | Production, Dev/Test  | Windows, Linux, Azure Arc |
| Boot Diagnostics              | Production, Dev/Test  | Windows, Linux            |
| Windows Admin Center          | Production, Dev/Test  | Windows                   |
| Azure Automation Account      | Production, Dev/Test  | Windows, Linux, Azure Arc |
| Log Analytics Workspace       | Production, Dev/Test  | Windows, Linux, Azure Arc |

## Create a custom configuration profile

What if you don't want to use all the features and services in the built-in profiles? You just make your own custom profile. Here's how: Go to the Automanage service and click the "Configuration profiles" blade. Then click the Create button in the top left of the blade.

Here you can start checking and unchecking the features you want to include in the configuration profile. You can also configure a backup policy and Antimalware scan policy. As you might notice, some of the services you can't check or uncheck. These are Azure Security Center, Automation Account, and Log Analytics Workspace. Azure Security Center is always checked, and will onboard you VMs to the free tier of the service. With Azure Automation Account and Log Analytics Workspace it will be checked or unchecked based on the other services you enable. For example, if you enable Change Tracking, it will also enable the Automation Account because it is dependent of one.

![custom profile](/img/Custom-Profile.PNG)