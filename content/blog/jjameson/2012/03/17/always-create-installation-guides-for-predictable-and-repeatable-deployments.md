---
title: "Always create installation guides for predictable and repeatable deployments"
date: 2012-03-17T22:37:26-07:00
excerpt: "Does your team utilize a step-by-step installation guide to build and deploy various environments, or do the Development, Test, and Release Management folks simply \"wing it\"? I certainly hope it's not the latter."
draft: true
categories: ["My System", "SharePoint"]
tags: ["My System", "SharePoint 2010"]
---

An important deliverable on most projects I work on is some form of installation
or deployment guide. For example, I typically provide clients with a document
that details the step-by-step process of installing and configuring SharePoint
Server 2010 based on their specific needs. Many projects involve multiple installation
guides (for example, an installation guide for the initial release of a solution,
followed by some sort of "upgrade" guide for subsequent releases).

Unfortunately, I've also been involved in some projects where environments
are built with little or no documentation (despite strong objections on my part).
For example, on the last project I worked on before I left Microsoft, the customer's
Development/IT teams built the various SharePoint VMs with multiple instances
of SQL Server installed locally and what seemed like every SharePoint service
enabled (at least that was the case on the VMs I had access to). Those development
VMs were excruciatingly slow at times and I finally ended up disabling a bunch
of "junk" on the VM assigned to me (e.g. extraneous SQL Server instances and
SharePoint services) just so I could get my work done.

On a related topic, I'm not sure whose idea it was at Microsoft to have the
checkboxes for every service in the Farm Configuration Wizard be checked *by default*. If I had been a member of the SharePoint product group, I would
have vehemently fought against that default configuration. While I agree the
Farm Configuration Wizard is useful for some scenarios (e.g. demo environments
and for small organizations that want to get up-and-running with minimal effort),
I still think it would have been much smarter to leave all of the checkboxes
cleared by default.

Wouldn't that also have been more inline with Microsoft's shift to provide
a minimal attack surface with the out-of-the-box configuration? I suppose one
could argue that since most of the SharePoint services are disabled by default,
you can't blame Microsoft if you mistakenly click through the Farm Configuration
Wizard and accidentally enable all 15 services (make that 14, as I was just
counting them, I realized the Lotus Notes Connector checkbox is not checked
by default.)

Note that much of the content in the installation guides that I provide is
included on TechNet. For example, the following section of TechNet provides
step-by-step procedures for installing SharePoint Server 2010 in various scenarios:

<cite>Deployment scenarios (SharePoint Server 2010)</cite>
[http://technet.microsoft.com/en-us/library/cc303424.aspx](http://technet.microsoft.com/en-us/library/cc303424.aspx)

Consequently you might be tempted to simply reference a few dozen TechNet
URLs in your installation guide and call it done. However I strongly recommend
against that for a number of reasons. First, the TechNet documentation is not
always correct. After all, this documentation is created by people and everyone
makes mistakes -- which leads to the second point. If you simply reference the
content on TechNet, how do you know when it has changed (either to fix mistakes
or when new content has been added)?

While you typically want to include updates and corrections in your installation
process, you need to make sure this is done according to *your* change
management process -- not Microsoft's. This will help ensure that all of your
environments are built and configured in a consistent manner.

In my next post, I will provide a sample installation guide for an extranet
solution based on SharePoint Server 2010 and Office Web Apps.

