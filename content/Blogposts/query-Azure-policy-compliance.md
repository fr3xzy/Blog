---
title: "Query-Azure-Policy-Compliance"
description: ""
date: "2022-05-06T14:48:39+02:00"
categories:
  - "Azure"
tags:
  - "Azure Policy"
  - "Governance"
#menu: main # Optional, add page to a menu. Options: main, side, footer
draft: true
type: "blog"
# Theme-Defined params
thumbnail: "img/placeholder.png" # Thumbnail image
lead: "How do you get more information about your Azure policy compliance?" # Lead text
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

Last blogpost I showed you an Azure Policy that checked for a SQL server firewall rule. The results would be a compliance view in Azure Policy. After the blogpost, me and Dennes Torres had a chat about the results you get in the compliance view. The problem: you don't get the resource ID or resource name for the SQL server with the setting on. This isn't really a problem if you have only one not-compliant resource, but if you have several, it becomes tedious.

The problem occours because the firewall rule is it's own resource. That makes the firewall rule and not the SQL server non-compliant. Therefore in the Azure policy compliance view, you only see the firewall rules. You also see the resource group that the resource is in, but again you could have several SQL servers in a single resource group.

Let's look at what I would need to do to find the SQL server name which has the non-compliant firewall rule:

![policy noncompliant view](/img/azure-ploicycomview.PNG)

In this scenario I have three different SQL servers in a single resource group. Two of them are non-compliant and one is compliant. The hard way to find the name of the SQL servers is to click into each one of these non-compliant resources in this view. Here I can find the resource ID of the non-compliant resource, but it's the firewall rule that is non-compliant. So I have to hover my mouse over or copy the resource ID to see that somewhere in that string, there is the SQL server name.

![sql server resource ID](/img/sqlserver-resourceid.png)

I wouldn't want to do this for all the non-compliant resources, even if it is just the two of them. So what is the alternative? The title of this blogpost has already spoiled the answer, but it's to query the Azure Resource Graph. Don't be alarmed, even if you haven't done this before, it is very simple.

A great way to query the resource graph if you aren't a champion in KQL, is to use the Azure Resource Graph Explorer. In the Graph Explorer you get several pre written queries you can run. And you also get a visual way to explore the API's on the side. Here you can search for what you are looking for or start clicking your way through the APIs.

We need to query the collection of API's called policyresources. And the type we are looking for is "microsoft.policyinsights/policystates" This will return all resources that is check by atleast one compliance policy. Next, we specify which policy and compliance state we want