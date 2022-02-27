---
title: "Use Run Command to run scripts on your VM"
description: "A deeper dive into the Run Command feature and showing examples on how to utilize it in automation"
date: "2022-02-02T20:25:52+01:00"
categories:
  - "Azure"
tags:
  - "Azure Compute"
  - "Azure Operations"
  - "Azure VM"
  - "Operations"
#menu: main # Optional, add page to a menu. Options: main, side, footer
draft: false
type: "blog"
# Theme-Defined params
thumbnail: "img/runcommand.PNG" # Thumbnail image
lead: "Manage your VM by running PowerShell script straight from the Azure Portal." # Lead text
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

I started my previous blog by introducing Azure Run Command as a feature to run Powershell and Bash scripts on VMs straight from the Azure Portal. In this blog I'm gonna dive deeper into the feature and show a real life example of how to use it.

## The two versions of Run Command

Something I didn't mention in my last blog is that there are actually two different versions of Run Command. They are action Run Command and Managed Run Command. Action Run Command is the most known version. This is also the feature that you have in the menu for VMs in Azure:

![Run command feature](/img/Azure-run-command.PNG)

## Action Run Command

Let's explore what action Run Command actually is so we can compare the two features better! To put it simply, action Run Command is just a API call. When you start a Run Command script the portal sends a POST request to this API:

```
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/runCommand?api-version=2021-07-01
```

And here's an example of what the body of the request looks like:

```json
{
"commandId": "RunPowerShellScript"
"script": "Your script here"
}
```

But how does that help us? Well, now that we know that it' a API call we could automate it. For example, let's say that we have a VM that's a webserver and we have a URL test in Azure Monitor that checks the website that the VM hosts. The URL test checks for timeouts and will trigger an alert if the respond time goes over a configured limit. The alert then triggers a Logic App that will send a POST request to the API with a Powershell script in the request body. The script will restart the IIS service on the VM in hope that it will solve the problem.

Maybe I'll write a blogpost about that solution one day, but for now look at these refrences on [how to integrate an Azure monitor alert with Logic Apps](https://docs.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-common-schema-integrations) and [how to send a Resource Graph API call with Logic Apps](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/azure-rest-api-through-logic-app/ba-p/2193956)

## Managed Run Command

How about the Managed Run Command then? First I should mention that Managed Run Command is in public preview. You should always be careful to use preview features in production. Preview features do not have any SLAs, they could be dropped instead of released and if they're released the price for the feature can be higher than it was in preview.

On to what it is. Managed Run Command is its own resource type and it removes most of the restrictions that were present for its predesessor. Now that it's a resource type we can use it for more advanced situations than Action Run Command. Let's look at a comparisson between the types from the [Microsoft Docs](https://docs.microsoft.com/nb-no/azure/virtual-machines/run-command-overview#compare-feature-support):

| Feature Support                   | Action RunCommand                                     | Managed RunCommand                                                   |
|-----------------------------------|-------------------------------------------------------|----------------------------------------------------------------------|
| ARM template                      | No                                                    | Yes                                                                  |
| Long running                      | 90 min limit                                          | customer specified timeout                                           |
| Execution account                 | System/root                                           | Customer specified user                                              |
| Multiple run commands             | No                                                    | Multiple in parallel or sequenced                                   |
| Large output                      | Limited to 4K                                         | Uploaded to append blob                                              |
| Progress tracking                 | No                                                    | Yes                                                                  |
| Async execution                   | Goal state/provisioning waits for script to complete | Customer specified async flag if provisioning waits for the script |
| Virtual machine scale set support | No                                                    | Yes                                                                  |
| SAS generation                    | No blob support                                       | Automated, CRP generates SAS for customer blobs and manages them     |
| Gallery (custom commands)         | Only built-in commandslds                             | Customer can publish scripts and share them                         |

As you can see, there are a lot of upgrades from Action Run Command to the new Managed Run Command. Because of these upgrades we can use this feature for other purposes. Some of these purposes might be running script during the creation of a VM, long running reporting script or creating scripts for sharing and reuse.

## Create a Managed Run Command script with Bicep

Since Managed Run Command is a resource type and as of February of 2022 there doesn't seem to be a way to deploy Managed Run Command from the Azure Portal, I'm going to show you how to deploy it through a Bicep template. I used code to deploy this code!

The script that I'm deploying does not matter as im just gonna show you how to deploy it with Bicep. So I'm just gonna fill in some filler script. For a full reference on which properties are available with the Run Command API check [this link.](https://docs.microsoft.com/en-us/azure/templates/microsoft.compute/virtualmachines/runcommands?tabs=bicep)

```
resource runcommand 'Microsoft.Compute/virtualMachines/runCommands@2021-07-01' = {
  name: 'HelloWorld'
  location: resourceGroup().location
  parent: VM1
  properties: {
    source: {
      commandId: 'HelloWorld'
      script: 'Write-host "Hello World"'
    }
  }
}
```

What I start with is declaring a resource and using the Run Commands API. The Run Command resource is a child resource under the virtual machine resource. Meaning that I need a parent resource. For the example I'll call that parent for VM1. The part that is more interesting is the source property. Here I need to specify what the script is. I have used a simple Hello World command as an example, but it is also possible to use "SciptUri" instead. That means that you can have a script made already and stored in a storage account and then reference where the script is stored. Much simpler than writing the script into the Bicep file.

*Tip* If you want to use the Bicep file as a reusable template use paramters for commandId and scriptUri so you can define what script you wanna run and where it's located.

You can use the Bicep code in your VM deployment template for doing post deployment configuration or by by referencing an existing VM in the Bicep file. Like this:

```
resource VM1 'Microsoft.Compute/virtualMachines@2021-07-01' existing = {
  name: 'VM1'
}
```

## Summary

In this blogpost I have described the two different versions of the Run Command feature and compared them. The two examples of how to use the features have given an insight into other options to use them other than just runnning a command through the portal. Writing this blog and diving deeper into the technical side of the feature has been exiting and it is just why I started this blog. I'm sure that this won't be the last time that I write about this feature as I continue to play with it on my own.
