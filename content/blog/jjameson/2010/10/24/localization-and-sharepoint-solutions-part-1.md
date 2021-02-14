---
title: "Localization and SharePoint Solutions, Part 1"
date: 2010-10-24T22:09:00+08:00
excerpt: "The primary goal for the current sprint on the project I'm working on is to localize the client portal previously developed for the United States so that it can be used by other regions around the world -- initially Spain, Mexico, Argentina, and a few..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:  
>   
> 
> [http://blogs.msdn.com/b/jjameson/archive/2010/10/25/localization-and-sharepoint-solutions-part-1.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/10/25/localization-and-sharepoint-solutions-part-1.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.


The primary goal for the current sprint on the project I'm working on is to localize the client portal previously developed for the United States so that it can be used by other regions around the world -- initially Spain, Mexico, Argentina, and a few Latin American countries. In other words, the bulk of the effort for this iteration involves extracting the English strings embedded throughout the solution into resource files.

Note that the client portal is built on Microsoft Office SharePoint Server (MOSS) 2007. [I know, I know...hopefully one of these days we'll devote a sprint to upgrading to SharePoint Server 2010 -- but until then, it's "business as usual" in my world ;-) ]

Note that I've been involved in a couple of other large MOSS 2007 solutions that needed to support localization -- specifically [Agilent Technologies](http://www.chem.agilent.com) and [KPMG](http://www.kpmg.com). Upon looking at these sites, you might assume they both leverage the same approach to localization. While that is certainly true at a high-level (i.e. both solutions utilize resource files), the actual localization implementation is rather different between those two sites.

The first question that you need to answer before creating a localized SharePoint solution is "How do you want the localization to work?" By that, I mean do you want all of the out-of-the-box SharePoint functionality to be localized in the various languages supported by your solution?

Regardless of your answer to this question, the basic approach you take for localization is essentially the same (e.g. extracting strings into resource files). However, the answer to this question ultimately determines *how* the resource files are actually used to display localized text.

In the case of Agilent Technolgies, the answer to this first question was that localized text was primarily intended for customers (i.e. external users of the site) -- meaning content authors and system administrators see English text when performing tasks like creating/approving pages and managing site settings. This was driven by the fact that a single MOSS 2007 farm was used to serve content for the United States, Japan, South Korea, China, Taiwan, etc. and Agilent personnel in the U.S. are primarily responsible for managing the site. (Content management activities are delegated to individuals within the various regions).

For KPMG, the answer was just the opposite -- regions across the world are responsible for creating and maintaining their own sites and thus content authors and system administrators prefer to see localized text when performing tasks like creating/approving pages and managing site settings.

In other words, the Agilent solution did not involve the use of any OS or SharePoint language packs and thus required "custom" localization functionality, whereas the KPMG solution followed the more typical approach of installing language packs and leveraging the "out-of-the-box" localization functionality.

The OOTB localization that I'm referring to is the fact that after you install one or more language packs for SharePoint, when you subsequently go to create a new site, the **Create Site **page (newsbweb.aspx) displays a dropdown list listing the available languages.

Let's suppose that you pick **Spanish **when creating a new site. The[SPWeb.Language](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.spweb.language%28v=office.12%29.aspx) property on this new site is set to 3082 (not 1034 as suggested by the MSDN Library). Subsequent page requests for anything under the Spanish site will have the [`Thread.CurrentThread.CurrentUICulture`](http://msdn.microsoft.com/en-us/library/system.threading.thread.currentuiculture.aspx) set to "es-ES" -- thus instructing the Resource Manager to look up Spanish resources (e.g. localized text).

Consequently, all you need to do is provide Spanish resource files and SharePoint (and the .NET Framework) will essentially do the rest for you (although not quite everything -- something I'll cover in a separate post).

Now, what if you don't want to install any OS or SharePoint language packs, but you still need to support localized text on your site? In other words, you want something similar to the Agilent scenario. In that case, the `Thread.CurrentThread.CurrentUICulture` will always be set to "en-US" (assuming you installed the English version of MOSS 2007, obviously) because all of the SharePoint sites you create will have the SPWeb.Language property set to 1033. Consequently, you can't just add, for example, Spanish resource files to your solution and expect everything to "just work."

For the Agilent solution, Ron Tielke -- one of my Microsoft colleagues on that project -- developed a custom [SPResourceExpressionBuilder](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.spresourceexpressionbuilder%28v=office.12%29.aspx) to look up localized text based on the [SPWeb.Locale](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.spweb.locale%28v=office.12%29.aspx) property, which -- unlike the SPWeb.Language property -- can be changed at will by site administrators (via the **Regional Settings** page). Other approaches are certainly possible. The key point to understand is that you have to use a custom localization solution, since the OOTB functionality inherent in using language packs is not available in this scenario.

For my current project, we are using the Spanish language packs for SharePoint and the OS (necessary for the underlying .NET Framework stuff). This is especially useful since we are leveraging things like the ASP.NET Login control (to provide the "log in" functionality) and the SharePoint Welcome control (to provide the "log out" functionality), and consequently we don't have to spend any effort localizing them.

Therefore *most* of the localization infrastructure is provided for us out-of-the-box. As I mentioned before, I'll cover what isn't provided in a separate post.


> **Update (2011-04-04)**
> 
> [Part 2 in this series](/blog/jjameson/2011/04/04/localization-and-sharepoint-solutions-part-2-a-k-a-the-currentuicultureswitcher-class) is now available.

