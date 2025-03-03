---
title: "Help Santa Secure His Naughty List"
description: ""
date: "2022-12-23T00:00:00+01:00"
categories:
  - "Azure"
tags:
  - "Festive Tech Calendar"
  - "Bicep"
  - "IaC"
  - "Security"
  - "Automation"
#menu: main # Optional, add page to a menu. Options: main, side, footer
draft: false
type: "blog"
# Theme-Defined params
thumbnail: "img/festive-thumbnail.png" # Thumbnail image
lead: "This year I am a part of the Festive Tech Calendar and this blogpost is about securing secrets in Bicep deployments, with a festive twist." # Lead text
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


Even on the North Pole they must adopt new technologies, and Santa has started using Bicep to deploy his list of this year's presents. Last year, Santa got a huge fine from the North pole Data Protection Authority after he committed the full naughty list into source control in a public repository. This year, Santa needs help with securing his secrets in Bicep templates during deployments!


Before I start the blogpost, I want to give a shoutout to the organizers of the Festive Tech Calendar for the amazing work they do! This year's Festive Tech Calendar are raising money for the charity [Missing people](https://www.missingpeople.org.uk/). A charity that helps find and reconnect missing persons with their loved ones. If you want to help in the fundraising, you can go to this [website](https://www.justgiving.com/fundraising/festivetechcalendar2022) and donate. Thank you!

Now, on to helping Santa before Christmas!

Santa has some Bicep templates that resides in a public repository on Github, because Santa loves Open Source. The templates are deployed to his Azure environment using a Github Actions workflow. The template deploys a vnet, a VM, and then runs Run Commands against the VM with a PowerShell script that creates a file with the names of the naughty children.

Let's look at how Santa's Bicep template looks like today:

![Santas bicep template today](/img/santa-today.PNG)

Oh my! This is worse than I imagined! Not only is Santa passing the naughty list into source control, but the admin credentials as well! We'll start by helping Santa secure the admin credentials first!


### Securing the credentials

If we look at the original template, we see that Santa has a module for the VM and passes them into the module as parameters. The best way to secure credentials used in deployments is to not have them in the template at all. How would we do that? The answer is to put the credentials into an Azure Key Vault as secrets and give the Service Principal for the workflow access to get the secret.

I have now created a Key Vault and added the credentials as secrets:
![santas key vault](/img/santa-keyvault.PNG)

The Key Vault also has an Access Policy that allows the Service Principal to get the secret.
So how do we get the credentials from the Key Vault into the Bicep template?

Luckily, it's simple. We reference the secrets as existing resources like this:
![VM template secured](/img/santa-vmsecured.PNG)

Now the credentials are not even shown in the template or deployment! Microsoft handles the reference in the background and the only thing we are pushing to source control is the name of the key vault and the name of the secrets. With that out of the way, let's move on to securing the naughty list.


### Securing the naughty list!

If we look back at the original main template, Santa declared the naughty list as a variable and added the names of naughty children directly into it. We would like to move the secret list out of the template and prevent it from being logged in Azure during deployment. Here's how we would do that.

First, we need to change it from being a variable to being a parameter. But we do not want it as a regular parameter, we want it to be *Secure()*. When a parameter is defined as secure, it will not be logged during deployment. Unfortunately, you cannot use the secure() function on arrays, so we will have to change it to a string. Here's how it will look:

![The secure parameter](/img/santa-secureparam.PNG)

And now that the naughty list is a parameter, we can define the parameters in the workflow.
Now, we are not going to just declare the naughty list in another file that lies in source control. We are going to use Github Actions Secrets. This way we will not show the list to anyone, as it is securely referenced in the workflow in Github just like we did with the Key Vault for credentials.

![Adding the secret to Github](/img/santa-githubsecret.PNG)

And the last thing we need to do is to update the workflow with the naughty list parameter that we get from configured secrets:

![Referencing Secrets in pipeline](/img/santa-pipeline.PNG)

### Summary

We have now helped Santa secure all the secrets in his deployment and he can without a worry prepare for Christmas Eve! 
For the admin credentials on the VM, we put the securely in a Key Vault, gave the Service Principal access to the secrets and referenced the secrets during deployment as existing resources. And for the naughty list we changed it from being a variable and array to a secure string parameter, then we created a Github Action secret, and finally referenced the secret in the Github Action workflow.