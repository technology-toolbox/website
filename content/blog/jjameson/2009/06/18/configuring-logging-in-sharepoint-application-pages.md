---
title: Configuring Logging in SharePoint Application Pages
date: 2009-06-18T21:08:00-06:00
description:
  In my previous post I showed how my simple, but highly effective approach to
  logging can be used with ASP.NET Web applications -- including Microsoft
  Office SharePoint Server (MOSS) and Windows SharePoint Services (WSS). Note
  that SharePoint application...
aliases:
  [
    "/blog/jjameson/archive/2009/06/18/configuring-logging-in-sharepoint-application-pages.aspx",
  ]
categories: ["My System", "SharePoint"]
tags: ["Simplify", "MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/06/18/configuring-logging-in-sharepoint-application-pages.aspx"
---

In my
[previous post](/blog/jjameson/2009/06/18/configuring-logging-in-asp-net-applications-and-sharepoint)
I showed how my
[simple, but highly effective approach to logging](/blog/jjameson/2009/06/18/a-simple-but-highly-effective-approach-to-logging)
can be used with ASP.NET Web applications -- including Microsoft Office
SharePoint Server (MOSS) and Windows SharePoint Services (WSS).

Note that SharePoint application (i.e. \_layouts) pages are served from a
different virtual directory than content pages within the Web application.
Consequently, in order to view log messages when, for example, activating a
feature (e.g. using
[http://fabrikam/en-US/Products/_layouts/ManageFeatures.aspx](http://fabrikam/en-US/Products/_layouts/ManageFeatures.aspx))
the
[System.Web.WebPageTraceListener](http://msdn.microsoft.com/en-us/library/system.web.webpagetracelistener.aspx)
must be specified in the Web.config file within the following folder:

> %ProgramFiles%\Common Files\Microsoft Shared\Web Server
> Extensions\12\template\layouts

Also be sure to enable tracing using the `<trace>` element within `<system.web>`
in that Web.config file, as shown in my previous post.

Now you can easily view log messages from a feature receiver that uses the
custom `Logger` class regardless of whether the feature is "activated" via
**Site Settings**, through stsadm.exe, or from within
[Visual Studio via a unit test](/blog/jjameson/2007/03/22/what-s-in-a-name-defaultfeaturereceiver-vs-featureconfigurator).
