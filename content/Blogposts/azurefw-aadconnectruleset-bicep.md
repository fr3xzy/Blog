---
title: "Deploy Azure Firewall ruleset for Azure AD with Bicep"
description: "Get the required firewallrules in Azure firewall for Azure AD connect using a Bicep template."
date: "2022-03-23T13:57:21+01:00"
categories:
  - "Azure"
tags:
  - "Bicep"
  - "Azure Firewall"
  - "IaC"
  - "Azure AD Connect"
#menu: main # Optional, add page to a menu. Options: main, side, footer
draft: false
type: "blog"
# Theme-Defined params
thumbnail: "img/AAD-connect.PNG" # Thumbnail image
lead: "Get the required firewallrules in Azure firewall for Azure AD connect using a Bicep template." # Lead text
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

Anyone who have used the portal to add rules to Azure Firewall will know how tedious and annoying it can get. With small fields to write both destination and source, it does get old very quick. And something that I found myself doing for several customers was setting the rules required for Azure AD Connect. I quickly got tired of doing it manually, so I created a Bicep template to deploy it much easier and faster.

**Note** That this blogpost is about deploying the firewall rules to Azure Firewall. If you are only looking for required URLs go [here](#URLlist)

## Scenario

In today's scenario, we are talking about having a domain controller on a VM in Azure and using an Azure Firewall to manage in- and outbound traffic. We then want to sync out AD identities to Azure AD using Azure AD Connect.

![Scenario](/img/AAD-connect.PNG)

## Required URLs {#URLlist}

| URL                                 | Protocol:Port | Description                                         |
|-------------------------------------|---------------|-----------------------------------------------------|
| *.aadconnecthealth.azure.com        | https:443     | AAD Connect Health                                  |
| *servicebus.windows.net             | https:443     | AAD Connect Health                                  |
| *.adhybridhealth.azure.com          | https:443     | AAD Connect Health                                  |
| www.management.azure.com            | https:443     | Various Azure services                              |
| www.policykeyservice.dc.ad.msft.net | https:443     | AAD Connect Health                                  |
| secure.aadcdn.microsoftonline-p.com | https:443     | MFA                                                 |
| aadcdn.msftauth.net                 | https:443     | MFA                                                 |
| www.office.com                      | https:443     | Authentication                                      |
| *.windows.net                       | https:443     | Azure AD Graph                                      |
| *.microsoftonline.com               | https:443     | Configure Azure AD directory and import/export data |
| *.msappproxy.net                    | https:443     | Authentication                                      |
| *.verisign.com                      | http:80       | Download CRL lists                                  |
| *.entrust.net                       | http:80       | Download CRL lists                                  |
| *.crl3.digicert.com                 | http:80       | Verify certificates                                 |
| *.crl4digicert.com                  | http:80       | Verify certificates                                 |
| *.ocsp.digicert.com                 | http:80       | Verify certificates                                 |
| *.www.d-trust.net                   | http:80       | Verify certificates                                 |
| *.root-c3-ca2-2009.ocsp.d-trust.net | http:80       | Verify certificates                                 |
| *.crl.microsoft.com                 | http:80       | Verify certificates                                 |
| *.oneocsp.microsoft.com             | http:80       | Verify certificates                                 |
| *.ocsp.msocsp.com                   | http:80       | Verify certificates                                 |

If you can't get Azure AD Connect to work with these ports opened, try this [troubleshooting guide from Microsoft on the bare minimum requirements](https://docs.microsoft.com/en-us/azure/active-directory/hybrid/tshoot-connect-connectivity#troubleshoot-connectivity-issues-in-the-installation-wizard)

## Bicep template

```xml
param fwpolicyname string
param priority int = 300
param sourceaddresses array

resource fwpolicy 'Microsoft.Network/firewallPolicies@2021-05-01' existing = {
  name: fwpolicyname
}

resource fwPolicyrulecollection 'Microsoft.Network/firewallPolicies/ruleCollectionGroups@2021-05-01' = {
  name: 'AllowAadConnect'
  parent: fwpolicy
  properties: {
    priority: priority
    ruleCollections: [
      {
        ruleCollectionType: 'FirewallPolicyFilterRuleCollection'
        action: {
          type: 'Allow'
        }
        rules: [
          {
            name: 'AAD Connect health Service Endpoints'
            ruleType: 'ApplicationRule'
            targetFqdns: [
              '*.aadconnecthealth.azure.com'
              '*servicebus.windows.net'
              '*.adhybridhealth.azure.com'
              'www.management.azure.com'
              'www.policykeyservice.dc.ad.msft.net'
              'secure.aadcdn.microsoftonline-p.com'
              'aadcdn.msftauth.net'
            ]
            sourceAddresses: sourceaddresses
            protocols: [
              {
                protocolType: 'Https'
                port: 443
              }
            ]
          }
          {
            name: 'Discovery'
            ruleType: 'ApplicationRule'
            targetFqdns: [
              'www.office.com'
            ]
            sourceAddresses: sourceaddresses
            protocols: [
              {
                protocolType: 'Https'
                port: 443
              }
            ]
          }
          {
            name: 'Sign in AAD'
            ruleType: 'ApplicationRule'
            targetFqdns: [
              '*.windows.net'
            ]
            sourceAddresses: sourceaddresses
            protocols: [
              {
                protocolType: 'Https'
                port: 443
              }
            ]
          }
          {
            name: 'Confidure and import/export AAD'
            ruleType: 'ApplicationRule'
            targetFqdns: [
              '*.microsoftonline.com'
            ]
            sourceAddresses: sourceaddresses
            protocols: [
              {
                protocolType: 'Https'
                port: 443
              }
            ]
          }
          {
            name: 'Auth'
            ruleType: 'ApplicationRule'
            targetFqdns: [
              '*.msappproxy.net'
            ]
            sourceAddresses: sourceaddresses
            protocols: [
              {
                protocolType: 'Https'
                port: 443
              }
            ]
          }
          {
            name: 'CRL'
            ruleType: 'ApplicationRule'
            targetFqdns: [
              '*.verisign.com'
              '*.entrust.net'
            ]
            sourceAddresses: sourceaddresses
            protocols: [
              {
                protocolType: 'Http'
                port: 80
              }
            ]
          }
          {
            name: 'Verify cert'
            ruleType: 'ApplicationRule'
            targetFqdns: [
              '*.crl3.digicert.com'
              '*.crl4.digicert.com'
              '*.ocsp.digicert.com'
              '*.www.d-trust.net'
              '*.root-c3-ca2-2009.ocsp.d-trust.net'
              '*.crl.microsoft.com'
              '*.oneocsp.microsoft.com'
              '*.ocsp.msocsp.com'
            ]
            sourceAddresses: sourceaddresses
            protocols: [
              {
                protocolType: 'Http'
                port: 80
              }
            ]
          }
        ]
      }
    ]
  }
}
```

## Deploying the template

So how do you deploy this Bicep template? This template requires you to have an already configured Azure Firewall and a firewall policy. Next you need to configure the parameters specified at the top of the template. Only 2 of them are strictly necessary to configure as the priority parameter has a default value of 300, but if you have a rule with that priority, you need to change that too. The parameter "fwpolicyname" is as simple as the name of your existing firewall policy that you are deploying the ruleset to.

The "sourceaddresses" parameter is a bit different as it expects an array. Therefore, your input should look something like this:
```xml
[
  "10.10.7.5"
  "10.10.7.6"
]
```
It's possible to put several or only one IP-address depending on how many servers you have installed Azure AD Connect on.

I would recommend to use a smiple parameter file if you want a simpler experience when deploying. A parameter file would look like this:
```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "soureceaddresses": {
            "value": [
              "10.10.7.5"
              "10.10.7.6"
            ]
        },
        "fwpolicyname": {
          "value": "fwpolicyname111"
        },
        "priority": {
          "value": 250
        }
    }
}
```
Now to deploy it with Powershell, you use the following command (after authentication ofcourse):

```Powershell
New-AzResourceGroupDeployment -TemplateFile fwruleset.bicep -TemplateParameterFile parameters.json
```