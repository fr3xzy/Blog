---
title: "Update management with Azure Automation"
description: "How to keep your Azure VMs up to date with centralized management in Azure"
date: "2022-02-27T13:13:44+01:00"
categories:
  - "Azure"
tags:
  - "Azure Automation"
  - "Automation"
  - "Azure Compute"
  - "Azure Operations"
#menu: main # Optional, add page to a menu. Options: main, side, footer
draft: false
type: "blog"
# Theme-Defined params
thumbnail: "img/azureautomation.PNG" # Thumbnail image
lead: "How to keep your Azure VMs up to date with centralized management in Azure." # Lead text
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


With information security coming into the spotlight of mainstream media after more and more big companies experience millions of dollars in revenue loss due to security breaches and ransomware. Security is becoming, if not already; the field of IT with the most attention at it. Both companies and individuals are tearing apart operating systems and applications alike trying to find and even exploit the vulnerabilities they find. All this have made it even more critical for IT departments to be fast and thorough with patching systems because they are always catching up to the vulnerabilities.

If you ask an IT Pro how you should go about patching your systems, he might tell you that first you need to deploy to the test environment and check that it doesn't break anything, and maybe even wait a few days for the manufacturer to release a list of known problems with the patch. Then you can finally deploy it to production.

Even though that process lowers the risk of the patch breaking anything, it's not fast. If we look back on the Log4J vulnerability that was disclosed to the public on the 1st of December 2021, already on the 10th of December there were [reports](https://unit42.paloaltonetworks.com/apache-log4j-vulnerability-cve-2021-44228/#log4j-exploitation-attempts) of mass exploits of the vulnerability in the wild.

To manage patching of servers, IT-departments often have systems or third-party solutions that they can use to find and install new patches. Let's make up a scenario where you use SCCM to manage patching of your servers on-premises. And to support SCCM you also have 2-3 Software Update Point servers in different datacenters to minimize network traffic. Now, you have started a migration project to the cloud and you make an estimation on cost of lift and shift for your SCCM solution. You then see quite clearly that compute is expensive in the cloud. What do you do then?

Luckily Microsoft values security and offers a free solution in Azure called Update Management with Azure Automation. You now have a SaaS solution that you can use to replace the patch management part of SCCM.

## Managing patching with Update Management

There are different ways to enable Update Management on your VMs. Let's look at how to do it from the Virtual Machine overview in Azure. First go to the Azure portal and select Virtual Machines from the menu to the left:

![virtualmachine](/img/menu-vms.png)

Here you mark the VMs that you want to enable it for. Then click the three dots in the top menu. In the menu that pops up, click on services and finally Update Management.

![enableupdatemanagement](/img/vm-enable-um.png)

Here you choose to let Azure automatically configure a Log analytics workspace and automation account for you or choose already existing resources. You also get a summary that shows a status whether your VMs support Azure Update Management. If all the chosen VMs are supported, you now need to click enable.

![configureupdatemanagement](/img/updatemanagement-configure.PNG)

After clicking enable, the deployment will begin. This might take some time as Azure need to install the required extension on your VMs and register the machine as ready.

When your machines are done onboarding to Update Management, you will get a nice dashboard with your machines compliance. This will give you a nice overview of which machines are missing what updates. If you then need to install some of these updates you can click on "schedule update deployment". There you specify what updates you want installed and exclude certain types or specified patches. You can also specify other steps in the maintenance windows like if the servers should reboot and if there should be done some extra pre- or post-configuration on the VMs.

## Your options for automated patching in Azure

I have talked mostly about Update Management with Azure Automation, but it's not your only options for automating security updates on your VMs in Azure. There is a feature that completely automate it for you. With automatic VM guest patching you just have to enable the feature on your VMs and Azure will automatically install all critical and security patches.

Now there are both positives and negatives about automating every aspect of patching, but even with the tradeoffs, it might just be what you need. The big upside is obviously that it's fully automated from start till finish. On the downside, you are giving up all control. You don't get to choose what updates get installed or when they are installed.

Beacause Azure automatically installs all patches that are marked as critical or security. You don't get the chance to opt out of any of these patches. Maybe the patch brings in a bug that breaks printing? Or creates performance issues? So, if you feel fine about the tradeoff that you don't have any control, but then you don't need to think about it at all. Then Automated VM guest patching might be for you.
