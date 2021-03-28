---
title: "\"Failed to configure propagation share.\" Error Configuring Office SharePoint Server Search"
date: 2007-06-26T06:37:00-06:00
excerpt:
  Last weekend I had to rebuild our Test environment for Microsoft Office
  SharePoint Server (MOSS) 2007 to replace a VM with a physical server (for
  performance reasons ). During the rebuild, I encountered the following error
  when starting Office SharePoint...
aliases:
  [
    "/blog/jjameson/archive/2007/06/25/failed-to-configure-propagation-share-error-configuring-office-sharepoint-server-search.aspx",
    "/blog/jjameson/archive/2007/06/26/failed-to-configure-propagation-share-error-configuring-office-sharepoint-server-search.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/06/26/failed-to-configure-propagation-share-error-configuring-office-sharepoint-server-search.aspx"
---

Last weekend I had to rebuild our Test environment for Microsoft Office
SharePoint Server (MOSS) 2007 to replace a VM with a physical server (for
[performance reasons](http://blogs.msdn.com/jameson/archive/2007/06/24/performance-of-virtual-machines.aspx)).
During the rebuild, I encountered the following error when starting Office
SharePoint Server Search on the SSP server:

{{< div-block "errorMessage" >}}

> Failed to configure propagation share.

{{< /div-block >}}

Be aware that the code for configuring the share is not exactly robust --
meaning, if the share already exists then you'll get this error (even if the
existing share is configured exactly as it needs to be). To avoid the error, I
deleted the existing shares on the two front-end Web servers (previously
configured during the original build of the Test environment).

I suppose another option would have been to select the option to not configure
the share (I believe the option is something like "I'll configure the share
later using stsadm.exe"). However, deleting the share and having it recreated
(thus following the documented steps in our Install Guide) somehow made me feel
better. Go figure.
