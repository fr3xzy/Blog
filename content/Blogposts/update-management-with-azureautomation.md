---
title: "Update management with Azure Automation"
description: "How to keep your Azure VMs up to date with centralized management in Azure"
date: "2022-02-06T14:13:44+01:00"
categories:
  - "Azure"
tags:
  - "Azure Automation"
  - "Automation"
  - "Azure Compute"
  - "Azure Operations"
#menu: main # Optional, add page to a menu. Options: main, side, footer
draft: true
type: "blog"
# Theme-Defined params
thumbnail: "img/placeholder.png" # Thumbnail image
lead: "How to keep your Azure VMs up to date with centralized management in Azure." # Lead text
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


With informeation security comming into the spotlight of mainstream media after more and more big companies experience millions of dollars in revenue loss due to security breaches and ransomware. Security is becomming, if not already; the field of IT with the most attention at it. Both companies and individuals are tearing apart operatingsystems and applications alike trying to find and even exploit the vulnerabilities they find. All this have made it even more critical for IT departments to be fast and thorough with patching systems, cause they are always catching up to the vulnerabilities.

If you ask a IT Pro how you should go about patching your systems, he might tell you that first you need to deploy to the test environment and check that it doesn't break anything, and maybe even wait a few days for the manufactorer to release a list of known problems with the patch. Then you can finally deploy it to production.

Even though that process lowers the risk of the patch breaking anything, it's not fast. If we look back on the Log4J vulnerability that was disclosed to the public on the 1st of December 2021, already on the 10th of December there were [reports](https://unit42.paloaltonetworks.com/apache-log4j-vulnerability-cve-2021-44228/#log4j-exploitation-attempts) of mass exploits of the vulnerability in the wild.

To manage patching of servers, IT-departments often have systems or thirdparty solutions that they can use to find and install new patches. Let's make up a scenario where you use SCCM to manage patching of your servers on-premises. And to support SCCM you also have 2-3 Software Update Point servers in different datacenters to minimize network traffic. Now, you have started a migration project to the cloud and you make an estimation on cost of lift and shift for your SCCM soltuion. You then see quite clearly that compute is expensive in the cloud. What do you do then?

Luckily Microsoft values security and offers a free solution in Azure called Update Management with Azure Automation. You now have a SaaS solution that you can use to replace the patchmanagement part of SCCM.

## Managing patching with Update Management



## Your options for automated patching in Azure

I have talked mostly about Update Management with Azure Automation
