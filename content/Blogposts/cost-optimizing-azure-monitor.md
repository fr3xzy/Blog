---
title: "Cost Optimizing Azure Monitor"
description: ""
date: "2025-03-02T20:32:21+01:00"
categories:
  - "Azure"
tags:
  - "Cost Optimization"
  - "Azure Monitor"
  - "Governance"
  - "KQL"
  - "Azure Spring Clean"
#menu: main # Optional, add page to a menu. Options: main, side, footer
draft: true
type: "blog"
# Theme-Defined params
thumbnail: "img/placeholder.png" # Thumbnail image
lead: "Blogpost about cost optimizing Log Analytics monitoring for Azure Spring Clean 2025." # Lead text
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

Insight into your platform can be of huge benefit to both operations, developers and the security team, but logging to much can result in high costs. How can you optimize logging and monitoring to pay for only what you need. There are several ways to save on logging and some patterns and anti-patterns to be aware of to not overspend on logging. Let's look at that now.

Before we start, I would like to thank Thomas Thornton and Joe Carlyle who are once again hosting the Azure Spring Clean! And don't forget to check out the other amazing contributions to the community event over at the [Azure Spring clean website.](https://azurespringclean.com/)


## One or many workspaces?
The first question that you need to take into account when looking at optimizing your loggingstrategy is "Do I need all my logs in a central workspace?". There are some benefits to having all your logs in one workspace, and it used to be a recommended best-practice from Microsoft in the Azure Landing zones reference architecture. Having a central workspace makes it easy to have a single pane of glass for all your logs from the whole platform. It was also recommended to use this workspace for Sentinel. There was one big backside to this; it was expensive. When you activate Sentinel on a workspace, you double the cost for ingesting logs. Now, you would get the first 90 days of retention for free, compared to the 30 days with a regular workspace, but luckily there are now other features that can help us keep the first 90 days of retention low, but we will get to that.

So you now pay double for ingesting all your logs into the central workspace with Sentinel activated, but at least you can now have your security team simply monitor and query all the logs from the platform. Thats good right? Well, are all Azure logs vital or even related to security? No. Metrics and application insights logs are some of the larger log tables we have and usually they don't really matter for security. 

Another topic is that for Sentinel, you usually want a longer retention for those logs. Sometimes it's for regulatory reasons, other times for practicality. You probably don't need more than 3 months of metrics from your app service or no need to keep those application exemptions from 6 months ago in your logs? 

## Analyzing usage

## Basic logs and archive logs

## Move to resource specific logs

## Table Transformation

## Export for long-term storage

## Alerting 