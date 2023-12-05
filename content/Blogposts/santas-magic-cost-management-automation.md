---
title: "Santa´s Magic Cost Management Automation"
description: ""
date: "2023-12-05T00:00:00+01:00"
categories:
  - "Azure"
tags:
  - "Cost Management"
  - "Festive Tech Calendar"
  - "Automation"
  - "Governance"
#menu: main # Optional, add page to a menu. Options: main, side, footer
draft: false
type: "blog"
# Theme-Defined params
thumbnail: "/img/festive-thumbnail23.png" # Thumbnail image
lead: "My contribution for Festive Tech Calendar 2023" # Lead text
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

This blogpost is posted in correlation with the Festive Tech Calendar. Festive Tech Calendar is a community event that goes on through the whole of December. The event is raising donations for the Raspberry PI Foundation. The Raspberry PI Foundation is a charity that help children learn to code. Please checkout the [Just Giving page](https://www.justgiving.com/page/festive-tech-calendar-2023) and the [Festive Tech Calendar](https://festivetechcalendar.com/).

Santa's workshop has changed immensely the last 10 years. With kids wanting iPhones, PlayStations or the newest Fortnite battle pass. To adopt to this, half of the elves now work as developers from home, writing code for new cool games and digital toys. In the year 2023, AI is on the rise and Santa wants to capitalize on the hype and produce new cool toys using AI. To innovate new toys, the elves has gotten their own Azure Sandbox subscriptions to play around with the new services.

After 1 month, Santa realised that these AI services were a lot more expensive than he had anticipated. With the economy in recession, Santa needs to be careful with not overspending on development. To prevent the elves from using more than their projects allocated budget, Santa use Azure Cost Management to call automation for cleaning up the sandboxes. 

So Santa's solution is divided into 2 parts. He has created a budget and an action group that calls an Automation Account that runs a PowerShell script that deletes all resources in the sandbox subscription.

**Important** This demo of a PowerShell script will delete resources! Be very careful if you want to run it in your own environment. Please be sure that you understand and limit the damage it may cause.

This demo would require 2 different Azure subscriptions. 1 subscription that hosts the Automation Account and 1 subscription that will have all it's content deleted. A subscription in it self is free and therefore you shouldn't be afraid that cost will increase just because you need 2.

## The Automation Account

Let's start by looking at the Automation Account and the PowerShell script.
The Automation Account is placed in a seperate Azure subscription. A management subscription if you like. The Automation Account uses a system-assigned managed identity that has the role Contributor on the sandbox subscription. The Runbbok for the script is using PowerShell 7.1 and I have created a webhook so that we can call it from a action group. But don't think about that, you can set the webhook up when you configure the action group for the budget.

The PowerShell script it self is using a logic that I found in a cleanup script in the Azure Bicep CARML GitHub repo. The script is what Microsoft uses in CARML to cleanup resources that are deployed during moudle validation. The script is simple enough, it collects all resources and loops through all of them. It looks through what type the resource is and uses a switch to do special delete actions if they are needed. The default action of the switch is to just delete the resource. This script will only work with a single subscription. If you want to use it for potentially cleanup of several subscriptions based on which has a breached budget, you can use parameters and activate the common alert schema that can feed the info about which subscription has exceeded it's budget.

```PowerShell
## set variables
$resources = @()

## Auth
Connect-AzAccount -Identity

## Delete resources
$resources = get-azresources

foreach ($resource in $resources) {

    $type = $resource.type
    $ResourceId = $resource.Id

    switch ($type) {
        'Microsoft.Insights/diagnosticSettings' {
            $parentResourceId = $ResourceId.Split('/providers/{0}' -f $Type)[0]
            $resourceName = Split-Path $ResourceId -Leaf
            Remove-AzDiagnosticSetting -ResourceId $parentResourceId -Name $resourceName
            break
        }
        'Microsoft.Authorization/locks' {
            ## Skipping locks
            break
        }
        'Microsoft.Consumption/budgets' {
            ## Not deleting budgets
            break
        }
        'Microsoft.Insights/actionGroups' {
            ## Not deleting Action Group used by budget automation
            break
        }
        'Microsoft.KeyVault/vaults/keys' {
            $resourceName = Split-Path $ResourceId -Leaf
            Write-Verbose ('[/] Skipping resource [{0}] of type [{1}]. Reason: It is handled by different logic.' -f $resourceName, $Type) -Verbose
            # Also, we don't want to accidently remove keys of the dependency key vault
            break
        }
        'Microsoft.KeyVault/vaults/accessPolicies' {
            $resourceName = Split-Path $ResourceId -Leaf
            Write-Verbose ('[/] Skipping resource [{0}] of type [{1}]. Reason: It is handled by different logic.' -f $resourceName, $Type) -Verbose
            break
        }
        'Microsoft.ServiceBus/namespaces/authorizationRules' {
            if ((Split-Path $ResourceId '/')[-1] -eq 'RootManageSharedAccessKey') {
                Write-Verbose ('[/] Skipping resource [RootManageSharedAccessKey] of type [{0}]. Reason: The Service Bus''s default authorization key cannot be removed' -f $Type) -Verbose
            }
            else {
                Remove-AzResource -ResourceId $ResourceId -Force -ErrorAction 'Stop'
            }
            break
        }
        'Microsoft.Compute/diskEncryptionSets' {
            # Pre-Removal
            # -----------
            # Remove access policies on key vault
            $resourceGroupName = $ResourceId.Split('/')[4]
            $resourceName = Split-Path $ResourceId -Leaf

            $diskEncryptionSet = Get-AzDiskEncryptionSet -Name $resourceName -ResourceGroupName $resourceGroupName
            $keyVaultResourceId = $diskEncryptionSet.ActiveKey.SourceVault.Id
            $keyVaultName = Split-Path $keyVaultResourceId -Leaf
            $objectId = $diskEncryptionSet.Identity.PrincipalId

            Remove-AzKeyVaultAccessPolicy -VaultName $keyVaultName -ObjectId $objectId

            # Actual removal
            # --------------
            Remove-AzResource -ResourceId $ResourceId -Force -ErrorAction 'Stop'
            break
        }
        'Microsoft.RecoveryServices/vaults/backupstorageconfig' {
            # Not a 'resource' that can be removed, but represents settings on the RSV. The config is deleted with the RSV
            break
        }
        'Microsoft.Authorization/roleAssignments' {
            ## Don't want to remove roleAssignments
            break
        }
        'Microsoft.RecoveryServices/vaults' {
            # Pre-Removal
            # -----------
            # Remove protected VMs
            if ((Get-AzRecoveryServicesVaultProperty -VaultId $ResourceId).SoftDeleteFeatureState -ne 'Disabled') {
                Set-AzRecoveryServicesVaultProperty -VaultId $ResourceId -SoftDeleteFeatureState 'Disable'
            }

            $backupItems = Get-AzRecoveryServicesBackupItem -BackupManagementType 'AzureVM' -WorkloadType 'AzureVM' -VaultId $ResourceId
            foreach ($backupItem in $backupItems) {
                Write-Verbose ('Removing Backup item [{0}] from RSV [{1}]' -f $backupItem.Name, $ResourceId) -Verbose

                Disable-AzRecoveryServicesBackupProtection -Item $backupItem -VaultId $ResourceId -RemoveRecoveryPoints -Force
            }

            # Actual removal
            # --------------
            Remove-AzResource -ResourceId $ResourceId -Force -ErrorAction 'Stop'
            break
        }
        'Microsoft.OperationalInsights/workspaces' {
            $resourceGroupName = $ResourceId.Split('/')[4]
            $resourceName = Split-Path $ResourceId -Leaf
            # Force delete workspace (cannot be recovered)
            Write-Verbose ('[*] Purging resource [{0}] of type [{1}]' -f $resourceName, $Type) -Verbose
            Remove-AzOperationalInsightsWorkspace -ResourceGroupName $resourceGroupName -Name $resourceName -Force -ForceDelete
            break
        }
        'Microsoft.MachineLearningServices/workspaces' {
            $subscriptionId = $ResourceId.Split('/')[2]
            $resourceGroupName = $ResourceId.Split('/')[4]
            $resourceName = Split-Path $ResourceId -Leaf

            # Purge service
            $purgePath = '/subscriptions/{0}/resourceGroups/{1}/providers/Microsoft.MachineLearningServices/workspaces/{2}?api-version=2023-06-01-preview&forceToPurge=true' -f $subscriptionId, $resourceGroupName, $resourceName
            $purgeRequestInputObject = @{
                Method = 'DELETE'
                Path   = $purgePath
            }
            Write-Verbose ('[*] Purging resource [{0}] of type [{1}]' -f $resourceName, $Type) -Verbose
            $purgeResource = Invoke-AzRestMethod @purgeRequestInputObject
            if ($purgeResource.StatusCode -notlike '2*') {
                $responseContent = $purgeResource.Content | ConvertFrom-Json
                throw ('{0} : {1}' -f $responseContent.error.code, $responseContent.error.message)
            }

            # Wait for workspace to be purged. If it is not purged it has a chance of being soft-deleted via RG deletion (not purged)
            # The consecutive deployments will fail because it is not purged.
            $retryCount = 0
            $retryLimit = 240
            $retryInterval = 15
            do {
                $retryCount++
                if ($retryCount -ge $retryLimit) {
                    Write-Warning ('    [!] Workspace [{0}] was not purged after {1} seconds. Continuing with resource removal.' -f $resourceName, ($retryCount * $retryInterval))
                    break
                }
                Write-Verbose ('    [⏱️] Waiting {0} seconds for workspace to be purged.' -f $retryInterval) -Verbose
                Start-Sleep -Seconds $retryInterval
                $workspace = Get-AzMLWorkspace -Name $resourceName -ResourceGroupName $resourceGroupName -SubscriptionId $subscriptionId -ErrorAction SilentlyContinue
                $workspaceExists = $workspace.count -gt 0
            } while ($workspaceExists)
            break
        }

        Default {
            Remove-AzResource -ResourceId $ResourceId -Force -ErrorAction 'Stop'
        }
    }
}
```
## Budget setup

Now, on to the budget that we have created to trigger the runbook with the PowerShell script.
First we will add a simple budget for 500NOK:
![budget setup](/img/budget.png)

In the Set alerts tab we will set that when the actual cost of the subscription exeeds 100% of the configured budget, it will trigger the action group DeleteResources. This action group is calling the webhook of the runbook. To create the action group click the "manage action group" link right under alert conditions.
![budget alert](/img/budget-alert.png)

Create a new action group and give it the basic info such as subscription, resource group and a name. Notice that the Action group need to be in the same subscription as the budget is being configured for. Therefore, I have added functionallity to the script that it should skip action groups. In the Actions tab, chose that the desired action is to run an Azure Automation Runbook. In the dropdown menus that appear in the window, find and chose your Automation Account and Runbook. This will create the webhook that it needs to run the script on demand.
![budget action group](/img/budget-actiongroup.png)

That's how simple it is to clean up resources as a budget action. Hopefully this will help Santa to not overspend on the sandbox environments that he has setup for his elves. Again, don't forget to checkout [Festive Tech Calendar](https://festivetechcalendar.com/) to see all the awsome content that is being published throughout the whole of December!