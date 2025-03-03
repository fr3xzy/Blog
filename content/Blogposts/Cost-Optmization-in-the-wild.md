---
title: "Cost Optmization in the Wild"
description: "This is a blogpost where you find all code used in my presentation of Cost optimization in the wild."
date: "2023-06-12T22:27:19+02:00"
categories:
  - "Azure"
tags:
  - "Cost Optimization"
  - "Public Speaking"
  - "Governance"
  - "KQL"
  - "Azure Policy"
#menu: main # Optional, add page to a menu. Options: main, side, footer
draft: false
type: "blog"
# Theme-Defined params
thumbnail: "img/pandafindingcosts.png" # Thumbnail image
lead: "A collection of code used in presentation of Cloud Optimization in the wild" # Lead text
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

Here is a collection of snippets of code used in the "Cost Optimization in the wild! - Experiences from reducing costs" presentation.
If you are missing any snippets of code used in a demo, or if you have any other question, feel free to message me on Twitter or LinkedIn.

1. [Orphaned disks](#orphanedDisks)
2. [Old Snapshots](#oldsnapshots)
3. [App Gateways and Load Balancers](#balancers)
4. [Storage v1](#storagev1)
5. [App Service Plans](#appservices)

## Orphaned disks {#orphaneddisks}

### KQL to find disks

```SQL
resources
| where type == "microsoft.compute/disks" and properties.managedBy == ""
| where not(name endswith "-ASRReplica" or name startswith "ms-asr-" or name startswith "asrseeddisk-")
| project name,location,resourceGroup,subscriptionId,DiskSku=sku.tier,DiskSize=properties.diskSizeGB
```

## Old Snapshots {#oldsnapshots}

### KQL to find Old Snapshots 

```SQL
resources
| where type == "microsoft.compute/snapshots"
| where properties.timeCreated <= ago(60d)
| project name,location,resourceGroup,subscriptionId,diskSizeGB=properties.diskSizeGB,SKU=sku.name,Timecreated=properties.timeCreated
```

### Logic app to notify about Old snapshots
```json
{
    "definition": {
        "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
        "actions": {
            "HTTP_2": {
                "inputs": {
                    "authentication": {
                        "type": "ManagedServiceIdentity"
                    },
                    "body": {
                        "query": "resources | where type == 'microsoft.compute/snapshots' | where properties.timeCreated >= ago(60d) | project name,location,resourceGroup,subscriptionId,diskSizeGB=properties.diskSizeGB,SKU=sku.name,Timecreated=properties.timeCreated"
                    },
                    "headers": {
                        "Content-Type": "application/json"
                    },
                    "method": "POST",
                    "queries": {
                        "api-version": "2021-03-01"
                    },
                    "uri": "https://management.azure.com/providers/Microsoft.ResourceGraph/resources"
                },
                "runAfter": {},
                "type": "Http"
            }
        },
        "contentVersion": "1.0.0.0",
        "outputs": {},
        "parameters": {},
        "triggers": {
            "Recurrence": {
                "evaluatedRecurrence": {
                    "frequency": "Week",
                    "interval": 1
                },
                "recurrence": {
                    "frequency": "Week",
                    "interval": 1
                },
                "type": "Recurrence"
            }
        }
    },
    "parameters": {}
}
```

## Load Balancers and App GW {#balancers}

### KQL to find App Gateways without backend pools

```SQL
resources
| where type == "microsoft.network/applicationgateways"
| extend backendpool=parse_json(properties.backendAddressPools)
| mv-expand backendpool
| where backendpool.['properties.backendAddresses'] == ""
| project name,location,resourceGroup,subscriptionId,sku=properties.sku.name
```

### KQL to find Load Balancers without backend pools

```SQL
resources
| where type == "microsoft.network/loadbalancers" and  properties.backendAddressPools == "[]"
| extend pipId=parse_json(properties.frontendIPConfigurations[0].properties.publicIPAddress.id)
| project name,location,resourceGroup,subscriptionId,sku=sku.name,pipId
```

## Storage v1 {#storagev1}

### KQL to find Storage accounts with storage v1

```SQL
resources
| where type == "microsoft.storage/storageaccounts" and ['kind'] == "Storage"
| project name,location,resourceGroup,subscriptionId,kind
```

### Azure Policy to block storage v1

```json
"policyRule": {
  "if": {
    "allOf": [
      {
        "field": "type",
        "equals": "Microsoft.Storage/storageAccounts"
      },
      {
        "field": "kind",
        "equals": "Storage"
      }
    ]
  },
  "then": {
    "effect": "deny"
  }
}
```

## App service plans {#appservices}

### KQL to find App Service Plans without apps

```SQL
resources
| where type == "microsoft.web/serverfarms" and properties.kind != "functionapp" and properties.numberOfSites == 0
| project name,resourceGroup,location,subscriptionId,SKU=sku.name,kind
```

### PowerShell to find stopped web apps
```Powershell
$stoppedWAs = Get-AzWebApp | Where-Object {($_.State -EQ "Stopped") -and ($_.Kind -ne "functionapp")}

foreach($stoppedWA in $stoppedWAs) {
    $stoppedWAASP = Get-azappserviceplan | Where-Object Id -EQ $stoppedWA.ServerFarmId
  }
```