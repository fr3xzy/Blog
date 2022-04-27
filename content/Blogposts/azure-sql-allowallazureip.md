---
title: "The SQL Server AllowAllAzureIps setting"
description: ""
date: "2022-04-27T09:08:39+02:00"
categories:
  - "Azure"
tags:
  - "Azure PaaS"
  - "Azure SQL"
  - "Security"
  - "IaC"
  - "Bicep"
  - "Governance"
#menu: main # Optional, add page to a menu. Options: main, side, footer
draft: false
type: "blog"
# Theme-Defined params
thumbnail: "/img/sql-server-fw-setting.png" # Thumbnail image
lead: "The story about a deep dive into Azure SQL firewall rules and the setting AllowAllAzureIps" # Lead text
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

This is going to be a longer one. You have been warned!
This adventure started last week when I attended the Azure User Group Norway with a session on Azure SQL networking secrets by [Dennes Torres](https://twitter.com/Dennes). 

Dennes showed a setting in the firewall rules on Azure SQL server that said, "Allow Azure services and resources to access this server". Now you might think "Yeah, I need that for my App Service to access the database", but in reality, this setting will allow **ALL** Azure IPs to access the SQL server! Both me and the other attendees at the session was in shock and thought "How do I prevent this from being turned on?". This got me diving down the rabbit hole of firewall rules on Azure SQL server and how this setting worked.

In this blogpost I will go through my path from learning about this setting to creating a Bicep file for deploying a policy definition and assignment on subscriptions.

## Where to start?

My first thought in all of this was: "There's got to be a built-in Azure Policy for this". I searched through Azure Policy and found nothing. I went through all Azure Security Center recommendations to see if there were a policy definition there. Not a single mention of the AllowAllAzureIps setting.

## Searching through the APIs

A resource I often use when creating Bicep templates is the [resource reference](https://docs.microsoft.com/en-us/azure/templates/). There I can browse the APIs and see what properties are configurable on them. I browse through all the sub-APIs for SQL server and the SQL server API itself. Nothing...

I then set up a SQL server in my demo environment and start looking at it in Azure Resource Explorer, but still, I find nothing on the setting. It appears that this setting is configurable in the GUI, but not on the API. The more I dive deeper into this, the more curious I get!

## Finding the setting

The setting had to be there somewhere on the resource! So how do you find all settings configured on a resource if it's not on the API? It seems that my experience of creating Bicep templates helped me once again! If you export a resource to an ARM-template, you get all configured settings in the Json-file.

And there it was! It was there all along! 

![azure-sql-template](/img/azure-sql-fwtemplate.png)

It was set as a firewall rule on the API! Into the firewall rules I go! Now there was a Powershell command to get Azure SQL Server firewall rules, but there was one problem. You could only get the rules of one SQL server at a time.

![SQL server firewallrule Powershell command](/img/sql-server-fw-powershell.PNG)

I could script this to get the setting for all my SQL servers, but I wanted to prevent it from being set as well. And the only way to do that is with Azure Policy. But with the information we have so far, it should be easy. 

## Creating a Policy

The problem was that I had never created a custom Policy before. I had to find something I could copy, and not start from scratch. Back to the built-in policies I went. Luckily, I found exactly what I was looking for.

![SQL server Public Access policy](/img/sql-server-fw-publicaccess.PNG)

A policy that checked for a setting on a resource and then setting an effect. Now I just needed to change it up with the values I had and try it out!

Here's what the policy rule ended up looking like:
````json
{    
    "policyRule": {
      "if": {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Sql/servers/firewallrules"
          },
          {
            "field": "name",
            "like": "AllowAllWindowsAzureIps"
          }
        ]
      },
      "then": {
        "effect": "audit"
      }
    }
}
````

As you can see, I chose to start with an audit effect to see if I was on the right track. After deploying and assigning the policy definition I went on to check my Policy compliance. Now anyone who have worked with compliance in Azure Policy know that it is very slow to check the compliance. And if you don't know, here is the Powershell to force the compliance check:
````Powershell
Start-AzPolicyComplianceScan
````
After the long wait for the compliance check, it was clear that I had the correct policyrule. I then tested with the rule set with "Deny" effect. I think that was the first time I had ever been overflowing with joy and dopamine as my deployment failed! Finally, success!

## Creating a template for deployment

So, I had created one policy that checked if the setting was there and one that denied the setting from being set. Now I needed to create a simple way to deploy this in other Azure-environments. And as always, I turned to Bicep. This was the easy part!

I went on to create two different Bicep-templates, one for audit and one for denying. Of course, I could have created one template and used parameters to determine if it should audit or deny, but I knew that I was going to be sharing this with coworker and customers, and that means simplicity is essential.

If you want to use the templates for yourself, you can find them both [Here](https://github.com/Erlendrushf/Bicep/tree/main/Blog/SQLserver). And here is how the Deny template ended up looking like:

````xml
targetScope = 'subscription'
param location string = 'WestEurope'

resource sqldenypolicy 'Microsoft.Authorization/policyDefinitions@2021-06-01' = {
  name: 'Deny SQL AllowAllAzureIps'
  properties: {
    description: 'Deny SQL server from having AllowAllAzureIps configured.'
    displayName: 'Deny SQL server AllowAllAzureIps'
    mode: 'All'
    metadata: {
      category: 'SQL'
    }
    policyType: 'Custom'
    policyRule: {
      if: {
        allOf: [
          {
            field: 'type'
            equals: 'Microsoft.Sql/servers/firewallRules'
          }
          {
            field: 'name'
            like: 'AllowAllWindowsAzureIps'
          }
        ]
      }
      then: {
        effect: 'deny'
      }
    }
  }
}

resource policyass 'Microsoft.Authorization/policyAssignments@2021-06-01' = {
  name: 'Assign Deny AllowAllAzureIps Policy'
  location: location
  properties: {
    policyDefinitionId: sqldenypolicy.id
    description: 'Denies the use of setting "AllowAllAzureIps"'
    nonComplianceMessages: [
      {
      message: 'AllowAllAzureIps setting is forbidden.'
      }
    ]
    displayName: 'Deny SQL server AllowAllAzureIps'
  }
}
````

## Closing words

This deep dive is the most fun I have had learning in a long time. The roller coaster of not finding anything until suddenly you find the smoking gun, and everything falls into place. I would like to point out that I would not have managed to solve this if it hadn't been for my experience with templates and exploring the APIs in the process. Because of that, I recommend anyone that wants to become an Azure expert to look into the documentation on the Azure APIs and how to use them to query or configure resources.