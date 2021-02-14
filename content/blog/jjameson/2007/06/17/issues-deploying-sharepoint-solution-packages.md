---
title: "Issues Deploying SharePoint Solution Packages"
date: 2007-06-17T01:44:00+08:00
excerpt: "Several weeks ago, I converted our deployment process to use SharePoint solution packages instead of the batch scripts that we had been using in our Development environment. One of the issues that I discovered along the way is that SharePoint is rather..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
---

> **Note**
> 
> 
> 	This post originally appeared on my MSDN blog:
> 
> 
> 
> [http://blogs.msdn.com/b/jjameson/archive/2007/06/17/issues-deploying-sharepoint-solution-packages.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/06/17/issues-deploying-sharepoint-solution-packages.aspx)
> 
> 
> Since
> 	[I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog 
> 	ever goes away.


Several weeks ago, I converted our deployment process to use SharePoint solution  packages instead of the batch scripts that we had been using in our Development  environment. One of the issues that I discovered along the way is that SharePoint  is rather finicky when it comes to running the stsadm.exe command to deploy your  solution package.

I noted in a previous post that we have created a PublishingLayouts feature containing  our custom master pages, images, and stylesheets -- similar in structure to the  feature provided in Microsoft Office SharePoint Server (MOSS) 2007. Creating the  WSP file was quite straightforward, as was adding it to the solution store, using  the following command:



```
stsadm -o addsolution -filename Fabrikam.Project1.PublishingLayouts.wsp
```



However, as soon as I tried to deploy the solution using the following command:



```
stsadm -o deploysolution -name Fabrikam.Project1.PublishingLayouts -url
	http://foobar/ -local
```



I encountered the following error:


> This solution contains no resources scoped for a Web application and cannot 
> be deployed to a particular Web application.


I must have spent 30 minutes trying to figure out why this command did not work  (because it worked just fine for other features that I had converted to deploy with  WSPs). It turns out that I needed to omit the <kbd>url</kbd> parameter:



```
stsadm -o deploysolution -name Fabrikam.Project1.PublishingLayouts -local
```



The reason why the PublishingLayouts solution would not deploy with the <kbd>url</kbd> parameter is because, unlike the other features, there was  no assembly generated for the PublishingLayouts (since it was pure content).

I also encountered the following error when trying to deploy our custom Workflows  feature:


> Elements of type 'Workflow' are not supported at the 'WebApplication' scope. 
> This feature could not be installed.


I found that I had to omit the <kbd>url</kbd> parameter for this solution  as well.

I then decided to try omitting the <kbd>url</kbd> parameter when deploying  all of the other solutions. Without the <kbd>url</kbd> parameter, I was able  to deploy 7 of our 9 features. The remaining two produced the following error:


> This solution contains resources scoped for a Web application and must be deployed 
> to one or more Web applications.


For these two features, I *had* to specify the <kbd>url</kbd> parameter  when invoking stsadm.exe, because the manifest.xml file for the WSP specifies a `<SafeControl>` element. When  deploying these two solutions, SharePoint needs to know which Web.config file to  merge the `<SafeControl>` elements  into, and therefore the <kbd>url</kbd> parameter must be specified.

The bottom line is that if your solution specifies elements (a.k.a. "resources")  that need to be merged into a Web.config file (i.e. "for a Web application") then  you *must* specify the <kbd>url</kbd> parameter. If your solution does  not have an assembly or if your solution contains workflows, then you *cannot*  specify the <kbd>url</kbd> parameter.

