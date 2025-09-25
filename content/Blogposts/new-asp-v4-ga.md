---
title: "New App Service Plan v4 SKUs go GA"
description: ""
date: "2025-09-25T14:20:10+02:00"
categories:
  - "Azure"
tags:
  - "Cost Optimization"
  - "News"
  - "App Services"
#menu: main # Optional, add page to a menu. Options: main, side, footer
draft: false
type: "blog"
# Theme-Defined params
thumbnail: "img/appserviceplan.png" # Thumbnail image
lead: "Microsoft releases new App Service Plan SKU Family." # Lead text
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

In the early days of September 2025, Microsoft announced the general availability for their new v4 SKU family of the Azure App Service Plans. The new plan are based on the new v6 VMs that came GA earlier this year. The SKUs utilize new and better hardware that give big performance benefits. These new SKUs might help you realize some cost savings along the way.

Like the previous generation of App Serivce Plans (v3), there are both general purpose and memory optimized SKUs. This gives the flexability to get a better right-sizing for your applications needs. The difference now, is that with the new underlying hardware, you will get a lower cost to performance. So, even though the cost is the same per core, you will get a better performance for those cores. This will highly likely result in the possiblity to down-size some plans to a lower SKU tier. For example, from a P3v3 to a P2v4.

Let's look at the table that Microsoft has released on how much of a difference in performance you might experience with the new v4 SKUs.
Microsoft wrote the following about how they conducted the testing:
"The team ran comparative load tests for Premium v4 and Premium v3 with a classic ASP.NET version of the devshop site that has been used in App Service labs and presentations over the last few years.  The application is a simple classic ASP.NET site consisting of a few .aspx pages including a homepage, product category pages, and product detail pages.  The site makes no external calls to databases or cache servers. [...] Load tests were tuned to drive the underlying worker instance to around 80% CPU utilization, a value high enough to provide a good estimate of performance under load while still ensuring sufficient CPU headroom and not overloading the underlying VMs.  All tests used a single instance app service plan."

And here are the results:
![Table with results from load testing](/img/azure-asp-v4-loadtesting.png)

As you can see, the tests show a significant difference in performance. Therefore, you should look at the new App Service Plan v4 SKU family to see if you can reduce you costs by scaling down while still keeping the performance levels that your application requires.