---
title: "\"Failed to configure propagation share.\" Error Configuring Office SharePoint Server Search"
date: 2007-06-26T00:37:00+08:00
excerpt: "Last weekend I had to rebuild our Test environment for Microsoft Office SharePoint Server (MOSS) 2007 to replace a VM with a physical server (for performance reasons ). During the rebuild, I encountered the following error when starting Office SharePoint..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:  
>   
> 
> [http://blogs.msdn.com/b/jjameson/archive/2007/06/26/failed-to-configure-propagation-share-error-configuring-office-sharepoint-server-search.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/06/26/failed-to-configure-propagation-share-error-configuring-office-sharepoint-server-search.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.


Last weekend I had to rebuild our Test environment for Microsoft Office SharePoint Server (MOSS) 2007 to replace a VM with a physical server (for [performance reasons](http://blogs.msdn.com/jameson/archive/2007/06/24/performance-of-virtual-machines.aspx)). During the rebuild, I encountered the following error when starting Office SharePoint Server Search on the SSP server:


> Failed to configure propagation share.


Be aware that the code for configuring the share is not exactly robust -- meaning, if the share already exists then you'll get this error (even if the existing share is configured exactly as it needs to be). To avoid the error, I deleted the existing shares on the two front-end Web servers (previously configured during the original build of the Test environment).

I suppose another option would have been to select the option to not configure the share (I believe the option is something like "I'll configure the share later using stsadm.exe"). However, deleting the share and having it recreated (thus following the documented steps in our Install Guide) somehow made me feel better. Go figure.

