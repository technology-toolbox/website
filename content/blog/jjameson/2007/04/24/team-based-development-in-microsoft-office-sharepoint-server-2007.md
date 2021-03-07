---
title: "Team-Based Development in Microsoft Office SharePoint Server 2007"
date: 2007-04-24T07:56:00-06:00
excerpt: "Eric Charran has posted an article on MSDN with some great pointers on developing SharePoint solutions in a team environment: 
 
 http://msdn2.microsoft.com/en-us/library/bb428899.aspx 
 This is an interesting read and well worth the time spent. Overall..."
aliases: ["/blog/jjameson/archive/2007/04/23/team-based-development-in-microsoft-office-sharepoint-server-2007.aspx", "/blog/jjameson/archive/2007/04/24/team-based-development-in-microsoft-office-sharepoint-server-2007.aspx"]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2007/04/24/team-based-development-in-microsoft-office-sharepoint-server-2007.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/04/24/team-based-development-in-microsoft-office-sharepoint-server-2007.aspx)
>
> Since 			[I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that  			blog ever goes away.

Eric Charran has posted an article on MSDN with some great pointers on developing  	SharePoint solutions in a team environment:

> [http://msdn2.microsoft.com/en-us/library/bb428899.aspx](http://msdn2.microsoft.com/en-us/library/bb428899.aspx)

This is an interesting read and well worth the time spent. Overall I agree  	with the two approaches that Eric describes (i.e. assembly development vs. artifact  	development). However, at least in my opinion, certain aspects of the article  	really only apply to a "mature" MOSS development project.

Consider the scenario where you have just started a project developing your  	next generation Web platform on MOSS 2007. Since you are just starting out,  	you don't have DEV, TEST, and PROD environments setup yet.

Fortunately, each team member has a beefy desktop -- or a laptop with an  	external 7200 RPM USB drive ;-) -- and you have managed to find a server for  	the DEV (integration) environment. Therefore the VM approach described in the  	article is at least feasible to get things started.

However, the TEST and PROD environments won't be available for a couple of  	months (due to the need to procure more hardware, rack and cable the servers,  	perform security reviews since this is an externally-facing site, yadda, yadda,  	yadda).

In the beginning, you will likely rebuild the DEV environment a number of  	times (for example to create, refine, and verify the deployment process and  	the installation/configuration guide). The worst thing you could do -- again  	this is my opinion -- would be to consider DEV as a "production environment"  	(meaning a place "for business users to develop and deploy content") because  	doing so would constrain your Development team from effectively using "their"  	environment (i.e. rebuilding it at their discretion).

So, does that mean we need to delay the process of creating the master pages,  	style sheets, and page layouts until there is another, "production" environment?  	From a scheduling perspective, I certainly hope not.

Why not just create a "PublishingLayouts" feature like the MOSS team did  	and include your SharePoint Designer artifacts in there? In the beginning, these  	items utilize the same configuration management process as the "assembly development"  	(i.e. code) items. In other words, put these into version control -- thus ensuring  	they are not lost when someone decides to rebuild DEV. Is it kludgey? Yes, a  	little. As Eric mentions, SharePoint Designer has no integration with source  	control systems like TFS or VSS. However, saving these files to disk and then  	manually checking them into source control (at least in the beginning until  	there is a true "production" authoring environment) really isn't that big of  	a deal.

Using a feature makes it very easy to deploy these items to TEST and PROD  	as these environments become available. At some point, when you feel comfortable  	that your "production" environment is stable, you can certainly switch to using  	SharePoint as the "version control" system for the various content artifacts.

For most organizations, I recommend using TEST over the long term as your  	authoring environment for things like master pages, style sheets, and page layouts.  	You can then setup a content deployment job to "promote" these to PROD. This  	greatly simplifies the validation and verification aspects of these types of  	site changes. For many organizations, pure content changes (e.g. creating a  	new press release) can be done directly in PROD in order to facilitate a simple  	approval process for releasing content rather than requiring content deployment  	jobs. It all depends on how strict you need your content publishing process  	to be.

My $0.02 anyway...let me know your thoughts.

