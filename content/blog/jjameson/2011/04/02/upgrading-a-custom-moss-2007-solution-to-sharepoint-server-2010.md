---
title: "Upgrading a Custom MOSS 2007 Solution to SharePoint Server 2010"
date: 2011-04-02T22:59:00+08:00
excerpt: "In yesterday's post , I provided a custom SharePoint Server 2010 solution based on Dan Cederholm's sample site for the fictitious Tugboat Coffee company (from his book Handcrafted CSS : More Bulletproof Web Design ). 
 Since I had originally \"ported..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "SharePoint 2010", "Tugboat"]
---

> **Note**
> 
>             This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2011/04/03/upgrading-a-custom-moss-2007-solution-to-sharepoint-server-2010.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/04/03/upgrading-a-custom-moss-2007-solution-to-sharepoint-server-2010.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

In [yesterday's post](/blog/jjameson/2011/04/02/web-standards-design-with-sharepoint-part-4), I provided a custom SharePoint Server 2010 solution based         on Dan Cederholm's sample site for the fictitious Tugboat Coffee company (from his         book [Handcrafted CSS : More Bulletproof Web Design](http://amzn.com/0321643380)).

Since I had originally "ported" the Tugboat site to Microsoft Office SharePoint         Server (MOSS) 2007, most of the effort in getting the site into SharePoint 2010         was related to upgrading the custom Tugboat master page to support the SharePoint         ribbon. You can read more about that in my previous post.

However, I also took the time to migrate the previous Visual Studio 2008 solution         to Visual Studio 2010 in a way that leverages the new SharePoint development features         in VS2010 as much as possible. The following figure illustrates the old and new         solutions as well as the key mappings between the two.

![Upgrading the Tugboat solution from MOSS 2007 to SharePoint Server 2010](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/9/r_Tugboat-Solution-Upgrade.jpg)
Figure 1: Upgrading the Tugboat solution from MOSS 2007 to SharePoint Server
2010

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/9/o_Tugboat-Solution-Upgrade.jpg)

Here are the highlights from my experience upgrading the Tugboat project.

### "Buh-bye StsAdm.exe...Hello PowerShell"

Unless you've been living under a rock up to this point, I'm sure you know by now         that StsAdm.exe is no longer "in vogue." Instead you should be using PowerShell         whenever possible for configuring your SharePoint sites and deploying your solutions.

You can read more about my "DR.DADA" PowerShell scripts in a [previous blog post](/blog/jjameson/2011/02/27/deployment-scripts-for-sharepoint-server-2010).

### Logging custom messages and events to ULS

Prior to SharePoint Server 2010, I wasn't a big fan of the SharePoint Unified Logging         System. Sure, it was great for helping Microsoft Support diagnose problems in your         environment, but aside from that it seemed the only purpose of ULS was to "spew"         an unholy amount of [garbage](/blog/jjameson/2009/03/26/sharepoint-uls-logs-flooded-with-preserving-template-record-with-size) to your hard disk until it managed to completely consume all remaining         free space on the disk. Fortunately, after a number of patches and service packs,         ULS finally managed to "contain itself" and at least stop writing until the very         last byte on disk was gone.

Also, before [ULS Viewer](http://archive.msdn.microsoft.com/ULSViewer)         came along, trying to extract useful information from the ULS logs was typically         a painful process. Before adding ULS Viewer to my toolbox, I did use LogParser for         a while -- which at least made the task of "diving into the ULS logs" somewhat tolerable         -- but I still dreaded the times when things broke and I had to try to figure out         why.

Thankfully, ULS was substantially improved in SharePoint 2010. The product group         also made it much, much easier to write your own messages and events using ULS.         Now that you can specify your own categories, there's really no reason not to log         to ULS from your custom SharePoint solutions.

As you can see in the above mapping, my new **SPLogger **class essentially         replaced the old **Logger **class that I used in MOSS 2007. I should         probably cover the **SPLogger **class in a separate post, since some         of you are probably wondering why I wrote this code instead of using the **SharePointLogger
**class from [the SharePoint Guidance section on MSDN](http://msdn.microsoft.com/en-us/library/ff649628.aspx).

I'll try to get to that...eventually.

### Let Visual Studio 2010 manage your Feature.xml files and other XML "goo"

I've mentioned in the past that, even after tools like WSPBuilder came along, I         preferred to manage the process of deciding what to package in custom WSPs for MOSS         2007. Once you had a basic solution structure in place, creating new features and         adding various types of files to existing features was really just a matter of copy/paste.

However, I must say that I've really grown fond of the new SharePoint tooling support         in Visual Studio 2010. It's not perfect -- get over it, no software is ever perfect         -- and I still find myself having to frequently tweak the XML that is automatically         generated as a result of adding a file to the solution. However it's still a huge         step forward in terms of SharePoint developer productivity.

I think my biggest gripe is that by default when you <kbd>CTRL+Shift+B</kbd> to         build the Visual Studio solution, SharePoint projects don't automatically update         the WSP. In my experience, the time required to rebuild the WSP is short enough         that I'd prefer the "Package" option to be the default. I suppose I really should         just modify the project file to set the **IsPackaging **variable.

### "DefaultFeatureReceiver is dead, FeatureConfigurator is not"

Several years ago, I explained why I like to [separate the bulk of the code for custom **SPFeatureReceiver **classes](/blog/jjameson/2007/03/22/what-s-in-a-name-defaultfeaturereceiver-vs-featureconfigurator).         As you can see from the mapping above, you no longer need to create a **DefaultFeatureReceiver
**class -- the SharePoint features in VS2010 will easily create an equivalent         for you with a couple of clicks of the mouse.

Whether or not you choose to "refactor" the build of your feature configuration         code into a separate class is really up to you. Personally, I still find separate         "FeatureConfigurator" classes easier to debug.

### "Theme1" incremented to "Theme1.1"

In the [first part of my "Web standards design with SharePoint" series](/blog/jjameson/2010/01/30/web-standards-design-with-moss-2007-part-1), I explained         how I like to use "Theme" folders to organize CSS files and related images that         define a specific look-and-feel for a site. In a later post, I described how to         [avoid issues with caching by using "theme versions."](/blog/jjameson/2010/11/16/avoid-issues-with-caching-by-using-quot-theme-versions-quot)

Assume for the moment that Tugboat was a real site running on MOSS 2007. In that         case, we definitely would have [enabled disk-based caching](/blog/jjameson/2009/03/27/always-enable-disk-based-caching-in-moss-2007) (i.e. the BlobCache) to ensure SharePoint scaled         to support the thousands of coffee addicts visiting the site.

As described in [yesterday's post](/blog/jjameson/2011/04/02/web-standards-design-with-sharepoint-part-4), during the process of upgrading the solution to SharePoint         2010, I had to make a few tweaks to the CSS files in order to resolve some issues         specific to the new version of SharePoint.

Consequently, in order to force the updated CSS files to be downloaded the next         time customers visited the upgraded site, I simply renamed the folder and updated         the corresponding references throughout the solution. This is one of those areas         where the SharePoint tooling in Visual Studio 2010 isn't quite "perfect." While         it does update some of the attributes in the Elements.xml file (specifically, the         **Path **attributes), it doesn't make all of the necessary changes         (e.g. the **Url **attributes are not updated). This might be a result         of my tweaking of the Elements.xml file (for example, to add **Type="GhostableInLibrary"**         to each &lt;File&gt; element), but it seems like this should be taken care of automatically.

Fortunately, this isn't a scenario that occurs frequently.

Okay, time to wrap this up and have some pancakes with my daughter :-)

