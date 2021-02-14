---
title: "Configuring Logging in ASP.NET Applications (and SharePoint)"
date: 2009-06-18T14:38:00+08:00
excerpt: "This post continues on the original post for my simple, but highly effective approach to logging and the follow-up post which introduced configuring logging for console applications . 
 Obviously not all solutions are simple console-based applications..."
draft: true
categories: ["My System", "SharePoint", "Development"]
tags: ["Simplify", "MOSS 2007", "Core Development", "WSS v3", "Web Development"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/06/18/configuring-logging-in-asp-net-applications-and-sharepoint.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/06/18/configuring-logging-in-asp-net-applications-and-sharepoint.aspx)
> 
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

This post continues on the original post for my [simple, but highly effective approach to logging](/blog/jjameson/2009/06/18/a-simple-but-highly-effective-approach-to-logging) and the follow-up post which  introduced [configuring logging for console applications](/blog/jjameson/2009/06/18/configuring-logging-in-a-console-application).

Obviously not all solutions are simple console-based applications. With ASP.NET  Web services and applications -- including Microsoft Office SharePoint Server (MOSS)  and Windows SharePoint Services (WSS) -- you can still view log messages from the `Logger` class very easily on a per-request basis.

Note that the [System.Web.WebPageTraceListener](http://msdn.microsoft.com/en-us/library/system.web.webpagetracelistener.aspx) can be specified in Web.config to enable logging  to the ASP.NET tracing feature:

```
<system.diagnostics>
    <sources>
      <source name="defaultTraceSource" switchName="allTraceLevel">
        <listeners>
          <add name="webPageTraceListener" />
        </listeners>
      </source>
    </sources>
    <sharedListeners>
      <add type="System.Web.WebPageTraceListener, System.Web, Version=2.0.0.0,
Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a"
        name="webPageTraceListener" traceOutputOptions="None" />
    </sharedListeners>
    <switches>
      <add name="allTraceLevel" value="All" />
      <add name="verboseTraceLevel" value="Verbose" />
      <add name="infoTraceLevel" value="Info" />
      <add name="warningTraceLevel" value="Warning" />
      <add name="errorTraceLevel" value="Error" />
      <add name="offTraceLevel" value="Off" />
    </switches>
  </system.diagnostics>
```

Also note that ASP.NET tracing must be enabled using the `<trace>`  element within `<system.web>`:

```
<system.web>
    <trace enabled="true" pageOutput="false" requestLimit="50" localOnly="true" />
  </system.web>
```

Be sure to set the `requestLimit`  high enough to enable access to the page trace you are interested in, but also be  aware that you can easily clear the captured traces and then browse to the page  of interest again.

Once configured, browse to the page on the site. Then modify the URL to browse  to Trace.axd (e.g. [http://fabrikam-local/Trace.axd](http://fabrikam-local/Trace.axd))  to display the Trace Viewer.

![ASP.NET Trace Viewer](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_ASP.NET%20Trace%20Viewer.png)

    Figure 1: ASP.NET Trace Viewer

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_ASP.NET%20Trace%20Viewer.png)

Locate the request that you want to view log messages for and click the corresponding **View Details** link.

![ASP.NET Trace Viewer - Request Details](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_ASP.NET%20Trace%20Sample.png)

    Figure 2: ASP.NET Trace Viewer - Request Details

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_ASP.NET%20Trace%20Sample.png)

Notice that the [WebPageTraceListener](http://msdn.microsoft.com/en-us/library/system.web.webpagetracelistener.aspx) even formatted the warning message in red. How cool is  that?! [For all you SharePoint developers out there, compare this with "diving"  into the ULS logs to find an error or warning for a particular page request!]

Be aware that the Web.config file you use for SharePoint applications varies  depending on whether you want to view log messages for a content page or for an  application (i.e. \_layouts) page. This is covered in my [next post](/blog/jjameson/2009/06/18/configuring-logging-in-sharepoint-application-pages).

