---
title: "Use Run Command to run scripts on your VM"
description: ""
date: "2022-02-02T20:25:52+01:00"
categories:
  - "Azure"
tags:
  - "Azure Compute"
  - "Azure Operations"
  - "Azure VM"
  - "Operations"
#menu: main # Optional, add page to a menu. Options: main, side, footer
draft: true
type: "blog"
# Theme-Defined params
thumbnail: "img/placeholder.png" # Thumbnail image
lead: "Manage your VM by running PowerShell script straight from the Azure Portal." # Lead text
comments: false # Enable Disqus comments for specific page
authorbox: true # Enable authorbox for specific page
pager: false # Enable pager navigation (prev/next) for specific page
toc: false # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
sidebar: "right" # Enable sidebar (on the right side) per page
widgets: # Enable sidebar widgets in given order per page
  - "search"
  - "recent"
  - "taglist"
  - "social"
---

## Azure Run Command
I started my previous blog by introducing Azure Run Command as a feature to run Powershell and Bash scripts on VMs straight from the Azure Portal. In this blog I'm gonna dive deeper into the feature and show a real life example of how to use it.


## The two versions of Run Command
Something I didn't mention in my last blog is that there are actually two different versions of Run Command. They are action Run Command and Managed Run Command. Action Run Command is the most known version. This is also the feature that you have in the menu for VMs in Azure:

![Run command feature](/img/Azure-run-command.PNG)

## Action Run Command
Let's explore what action Run Command actually is so we can compare the two features better! To put it simply, action Run Command is just a API call. When you start a Run Command script the portal sends a POST request to this API: 
```{
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/runCommand?api-version=2021-07-01
```
And here's an example of what the body of the request looks like:

```
{
"commandId": "RunPowerShellScript"
"script": "Your script here"
}
```
But how does that help us? Well, now that we know that it' a API call we could automate it. For example, let's say that we have a VM thats a webserver and we have a URL test in Azure Monitor that checks the website that runs on the VM. The URL test checks for timeouts and will trigger an alert if the respond time goes over a configured limit. The alert then triggers a Logic App that will send a POST request to the API with a Powershell script in the request body. The script will restart the IIS service in hope that will solve the problem.

Demo with logic app and automatic respons to run command

## Managed Run Command
How about the Managed Run Command then? First I should mention that Managed Run Command is in public preview. You should always be careful to use preview features in production. Preview features do not have any SLAs, they could be dropped instead of released and if they're released the price for the feature can be higher than it was in preview.

On to what it is. Managed Run Command is its own service.