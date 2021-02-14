---
title: "Error Handling in MOSS 2007 Applications"
date: 2010-03-20T10:06:00+08:00
excerpt: "In my previous post , I described the enhancements to my original Logger class for logging exceptions in a consistent fashion. 
 While error handling in .NET console applications and ASP.NET Web applications is fairly straightforward, things get quite..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2010/03/20/error-handling-in-moss-2007-applications.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/03/20/error-handling-in-moss-2007-applications.aspx)
> 
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

In my [previous post](/blog/jjameson/2010/03/20/logging-exceptions-in-net-applications), I described the enhancements to my [original **Logger** class](/blog/jjameson/2009/06/18/a-simple-but-highly-effective-approach-to-logging) for logging exceptions in a consistent  fashion.

While error handling in .NET console applications and ASP.NET Web applications  is fairly straightforward, things get quite a bit more complicated when dealing  with solutions built with Microsoft Office SharePoint Server (MOSS) 2007 and Windows  SharePoint Services. This is because SharePoint comes out-of-the-box with its own  custom error handling infrastructure (although it is based on the core functionality  provided by ASP.NET).

While the OOTB error handling in MOSS 2007 and WSS works well for intranet solutions,  it's not so great for Internet-facing sites.

To understand why, consider a publishing site configured with **BlueBand.master** as the master page:

![Publishing site with BlueBand master page](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/9/r_Publishing%20Site%20-%20BlueBand%20Master%20Page.png)

    Figure 1: Publishing site with BlueBand master page

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/9/o_Publishing%20Site%20-%20BlueBand%20Master%20Page.png)

Now, similar to the example error demonstrated in my previous post, let's add  a Web Part to the home page that throws an exception:

```
using System;
using System.Web.UI.WebControls.WebParts;

using Fabrikam.Demo.CoreServices.Logging;

namespace Fabrikam.Demo.WebParts
{
    /// <summary>
    /// A sample Web Part used to demonstrate error handling (or lack thereof).
    /// </summary>
    public class UnhandledExceptionWebPart : WebPart
    {
        /// <summary>
        /// Raises the <see cref="System.Web.UI.Control.Load"/> event.
        /// </summary>
        /// <remarks>
        /// This method is overridden to thrown an unhandled exception.
        /// </remarks>
        /// <param name="e">An <see cref="System.EventArgs"/> object that
        /// contains the event data.</param>
        protected override void OnLoad(
            EventArgs e)
        {
            base.OnLoad(e);

            DoSomethingBad();
        }

        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Performance",
            "CA1804:RemoveUnusedLocals",
            MessageId = "divideByZero")]
        private static void DoSomethingBad()
        {
            int numerator = 1;
            int denominator = 0;

            try
            {
                int divideByZero = numerator / denominator;
            }
            catch (DivideByZeroException ex)
            {
                string errorMessage = "Something bad happened.";
                Logger.LogError(errorMessage);

                throw new InvalidOperationException(
                    errorMessage,
                    ex);
            }
        }
    }
}
```

As shown in the following screenshot, browsing to the home page now displays  the SharePoint error page.

![SharePoint error page](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/9/r_SharePoint%20Error%20Page.png)

    Figure 2: SharePoint error page

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/9/o_SharePoint%20Error%20Page.png)

If this were an intranet site and we were using the OOTB **default.master** (instead of **BlueBand.master**) then the out-of-the-box SharePoint  error page would probably be acceptable for most organizations. For an Internet-facing  site, however, we almost certainly don't want to show a page with a completely different  look-and-feel -- and even worse, a page that provides a link to a "**Web Parts
Maintenance Page**" as well as a link to "**Troubleshoot issues with
Windows SharePoint Services.**" These are completely meaningless to the general  public accessing the site.

So, how can we provide a better user experience?

Well, we could certainly add try/catch blocks throughout our solution and try  to handle various exceptions in our custom code as gracefully as possible. However,  that's problematic and it is unlikely to handle all of the possible scenarios.

If our solution were simply built on top of ASP.NET -- instead of SharePoint  -- we could simply create an error page and use the `<customErrors>`element in the Web.config file:

```
<customErrors defaultRedirect="/Error.aspx" mode="On" />
```

However, that doesn't work in SharePoint applications, because SharePoint has  its own error handling infrastructure that overrides any error page specified in  the `<customErrors>`element.

I've seen some blog posts that advocate using a custom **HttpModule**  to override the OOTB error handling in SharePoint. While that certainly works, I'm  not a fan of that approach, because it feels like using a 3-lb. sledge hammer to  assemble a fine piece of furniture (meaning that you can make it work, but you might  very well break something in the process).

Instead, the approach that I like to use is to hook into the **[Error](http://msdn.microsoft.com/en-us/library/system.web.ui.templatecontrol.error.aspx)**  event from a custom master page. After all, if we're developing  an Internet-facing Web site, aren't we going to utilize a custom master page for  all of the pages that are visible to the general public?

This way, we can display our own error page so that the general public doesn't  see messages like "Troubleshoot issues with Windows SharePoint
Services."

The following screenshot shows a custom error page fashioned after BlueBand.master:

![Custom SharePoint error page](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/9/r_Custom%20Error%20Page.png)

    Figure 3: Custom SharePoint error page

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/9/o_Custom%20Error%20Page.png)

Notice how the details of the error are not displayed to users (which probably  wouldn't be of much help to them anyway and, more importantly, might divulge sensitive  information). Instead, the details of the exception are logged to the **Application**  event log.

So how do we make this user experience a reality?

First, we need to add a little bit of code to our custom master page. Specifically,  in the page **Init** event, we wire up an event handler for the page **Error** event:

```
namespace Fabrikam.Demo.Publishing.Layouts.MasterPages
{
    [CLSCompliant(false)]
    public partial class FabrikamMinimal : System.Web.UI.MasterPage
    {
        private void Page_Error(
            object sender,
            EventArgs e)
        {
            Logger.LogError(this.Context.Error, this.Request.Url);
            this.Context.ClearError();

            this.Server.Transfer("/_layouts/Fabrikam/Error.aspx");
        }

        protected void Page_Init(
            object sender,
            EventArgs e)
        {
            // If an unhandled exception occurs, then SharePoint transfers the
            // request to /_layouts/error.aspx (via SPRequestModule). This
            // causes a couple of issues:
            //
            // 1) The SharePoint error.aspx page is hard-coded to use
            // simple.master and therefore looks like an OOTB SharePoint page
            // (i.e. the page is not branded like the rest of the site)
            //
            // 2) The SharePoint error.aspx page displays the following
            // message: "Troubleshoot issues with Windows SharePoint Services"
            // (which we obviously don't want to display on an Internet-facing
            // site)
            //
            // To avoid these issues, add a custom error handler for the page
            this.Page.Error += new EventHandler(Page_Error);
        }
    }
}
```

Note that I'm essentially following the approach outlined in the following KB  article:

<cite>How to create custom error reporting pages in ASP.NET by using Visual
C# .NET</cite>
[http://support.microsoft.com/kb/306355](http://support.microsoft.com/kb/306355)

In the **Page\_Error** method, I first log the exception using [my custom **Logger** class](/blog/jjameson/2010/03/20/logging-exceptions-in-net-applications). Then I call the **[HttpContext.ClearError](http://msdn.microsoft.com/en-us/library/system.web.httpcontext.clearerror%28VS.80%29.aspx)**  method to prevent the error from continuing  to the **Application\_Error** event handler (which would subsequently  invoke the SharePoint error handling infrastucture). Finally, I transfer the request  to a custom application page (Error.aspx) in the Layouts folder -- thus displaying  a friendly error message to the user, while still preserving the URL of the original  request (as opposed to a redirect, which would change the URL shown in the browser).

Note that since Error.aspx is handling an unexpected exception, the last thing  we want is another error to occur in the error page. Consequently, I use a very  simple ASP.NET page with nothing in the code-behind. In fact, I don't even want  to [use the master page configured for my SharePoint site](/blog/jjameson/2009/09/20/inheriting-the-master-page-from-the-current-site-context-in-moss-2007) -- in case the original  exception occurred as a result of some code in the master page. [I suppose you could  use a static HTML file if you are really paranoid about an exception occuring in  your error page. However, so far, I haven't found the need to do this.]

Here's the sample Error.aspx file that I created to capture the screenshot shown  in Figure 3:

```
<%@ Assembly Name="Fabrikam.Demo.Publishing, Version=1.0.0.0, Culture=neutral, PublicKeyToken=786f58ca4a6e3f60" %>

<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="Error.aspx.cs" Inherits="Fabrikam.Demo.Publishing.Layouts.ApplicationPages.ErrorPage" %>

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html dir="ltr">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta http-equiv="Expires" content="0" />
    <title>Error </title>
    <link rel="stylesheet" type="text/css" href="/Style%20Library/en-US/Core%20Styles/Band.css" />
    <link rel="stylesheet" type="text/css" href="/Style%20Library/en-US/Core%20Styles/controls.css" />
    <link rel="stylesheet" type="text/css" href="/Style%20Library/zz1_blue.css" />
    <link rel="stylesheet" type="text/css" href="/_layouts/1033/styles/core.css" />
    <style type="text/css">
        .zz1_logoLinkId_1
        {
            text-decoration: none;
        }
    </style>
</head>
<body class="body">
    <form runat="server">
    <table cellpadding="0" cellspacing="0" class="master">
        <tr>
            <td height="100%" class="shadowLeft">
                <div class="spacer">
                </div>
            </td>
            <td valign="top">
                <table cellpadding="0" cellspacing="0" width="100%" class="masterContent">
                    <tr>
                        <td colspan="2" class="authoringRegion">
                             
                        </td>
                    </tr>
                    <tr>
                        <td colspan="2">
                            <table cellpadding="0" cellspacing="0" width="100%">
                                <tr>
                                    <td colspan="4" class="topArea">
                                        <table id="zz1_logoLinkId" class="logo zz1_logoLinkId_2" cellpadding="0" cellspacing="0"
                                            border="0">
                                            <tr>
                                                <td>
                                                    <table cellpadding="0" cellspacing="0" border="0" width="100%">
                                                        <tr>
                                                            <td style="white-space: nowrap; width: 100%;">
                                                                <a class="zz1_logoLinkId_1" href="/Pages/default.aspx" accesskey="1">Fabrikam</a>
                                                            </td>
                                                        </tr>
                                                    </table>
                                                </td>
                                            </tr>
                                        </table>
                                    </td>
                                </tr>
                                <tr class="topNavContainer">
                                    <td>
                                         
                                    </td>
                                </tr>
                            </table>
                        </td>
                    </tr>
                    <tr>
                        <td width="100%" valign="top">
                            <div class="mainContainer">
                                <div class="pageTitle">
                                    Error
                                </div>
                                <div class="mainContent">
                                    <p class="description">
                                        We apologize, but an error occurred and your request could not be completed.</p>
                                    <p class="description">
                                        This error has been logged. If you have additional information that you believe
                                        may have caused this error, please contact technical support.</p>
                                </div>
                            </div>
                        </td>
                    </tr>
                </table>
            </td>
            <td height="100%" class="shadowRight">
                <div class="spacer">
                </div>
            </td>
        </tr>
    </table>
    </form>
</body>
</html>
```

I'm not really advocating that your custom error page be as complex as the one  I've shown here. This is just what I used in order to get the look-and-feel to be  very similar to BlueBand.master without spending a great deal of time converting  this to semantic markup.

For comparison purposes, here's a much simpler custom error page based on [a Web standards design](/blog/jjameson/2010/01/30/web-standards-design-with-moss-2007-part-1):

```
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="Error.aspx.cs"
 Inherits="Fabrikam.Demo.Publishing.Layouts.ApplicationPages.ErrorPage,
   Version=1.0.0.0, Culture=neutral, PublicKeyToken=786f58ca4a6e3f60" %>

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title>Error</title>
    <link rel="stylesheet" type="text/css" href="/Style Library/Fabrikam/Themes/Theme1/Fabrikam-Main.css" />
</head>
<body>
    <form runat="server">
    <div class="container_12">
        <div id="masthead">
            <div id="logo">
                <a href="/" title="Go to site home page">
                    <img src="/Style Library/Images/Fabrikam/Fabrikam-logo.png" />
                </a>
            </div>
        </div>
        <div class="grid_12">
            <h1>
                Error</h1>
            <p>
                We apologize, but an error occurred and your request could not be completed.</p>
            <p>
                This error has been logged. If you have additional information that you believe
                may have caused this error, please contact technical support.</p>
        </div>
    </div>
    </form>
</body>
</html>
```

Isn't that cleaner than that nasty ol' table-based layout?

Anyway...

Once we have our error page deployed to the Layouts folder and wired in via our  custom master page, the next step is to ensure that the call to the **Logger.LogError** method doesn't go completely unnoticed. In other words, we need to ensure  that we have a listener configured for the corresponding trace source.

Although I've covered some basics about configuring logging in ASP.NET applications  (and SharePoint) in a [previous post](/blog/jjameson/2009/06/18/configuring-logging-in-asp-net-applications-and-sharepoint), I didn't cover how to log to the Windows event log.

Let's assume that whenever an error occurs within our custom code, we want the  details to be written to the **Application** event log. Although you  might be tempted to simply use the [EventLogTraceListener](http://msdn.microsoft.com/en-us/library/system.diagnostics.eventlogtracelistener.aspx) included in the .NET Framework, there's a fundamental  problem that you might not be aware of: it doesn't work with SharePoint sites in  several very important scenarios.

To demonstrate this, I'll add the following section to the Web.config file for  the **Internet** zone of my sample Fabrikam site (which I've configured  for Forms-Based Authentication and anonymous access):

```
<system.diagnostics>
    <sources>
      <source name="defaultTraceSource" switchName="allTraceLevel">
        <listeners>
          <add name="eventLogTraceListener" />
        </listeners>
      </source>
    </sources>
    <sharedListeners>
      <add type="System.Diagnostics.EventLogTraceListener, System,
Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089"
        name="eventLogTraceListener"
        initializeData="Fabrikam Site">
        <filter type="System.Diagnostics.EventTypeFilter"
          initializeData="Information" />
      </add>
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

> **Important**
> 
> Notice that I use an `EventTypeFilter`to avoid writing debug (a.k.a. "verbose") messages to the
> event log.

With the **EventLogTraceListener** configured, as soon as I browse  to my site and trigger the unhandled exception, I'm presented with the OOTB SharePoint  error page shown in Figure 2. In other words, I've reverted back to the undesired  user experience.

Enabling the call stack and disabling custom errors in the Web.config file reveals  the following error:

```
[SecurityException: The source was not found, but some or all event logs could not be searched.  Inaccessible logs: Security.]
   System.Diagnostics.EventLog.FindSourceRegistration(String source, String machineName, Boolean readOnly) +665
   System.Diagnostics.EventLog.SourceExists(String source, String machineName) +104
   System.Diagnostics.EventLog.VerifyAndCreateSource(String sourceName, String currentMachineName) +75
   System.Diagnostics.EventLog.WriteEvent(EventInstance instance, Byte[] data, Object[] values) +156
   System.Diagnostics.EventLog.WriteEvent(EventInstance instance, Object[] values) +10
   System.Diagnostics.EventLogTraceListener.TraceEvent(TraceEventCache eventCache, String source, TraceEventType severity, Int32 id, String message) +107
   System.Diagnostics.TraceSource.TraceEvent(TraceEventType eventType, Int32 id, String message) +195
   Fabrikam.Demo.CoreServices.Logging.Logger.Log(TraceEventType eventType, String message) +211
   Fabrikam.Demo.CoreServices.Logging.Logger.LogError(Exception ex, Uri requestUrl) +773
   Fabrikam.Demo.CoreServices.Logging.Logger.LogError(Exception ex) +32
   Fabrikam.Demo.Publishing.Layouts.MasterPages.FabrikamMinimal.Page_Error(Object sender, EventArgs e) +59
   System.Web.UI.TemplateControl.OnError(EventArgs e) +8690682
   System.Web.UI.Page.HandleError(Exception e) +84
   System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint) +6776
   System.Web.UI.Page.ProcessRequest(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint) +242
   System.Web.UI.Page.ProcessRequest() +80
   System.Web.UI.Page.ProcessRequestWithNoAssert(HttpContext context) +21
   System.Web.UI.Page.ProcessRequest(HttpContext context) +49
   ASP.WELCOMESPLASH_ASPX__533642652.ProcessRequest(HttpContext context) +4
   Microsoft.SharePoint.Publishing.TemplateRedirectionPage.ProcessRequest(HttpContext context) +184
   System.Web.CallHandlerExecutionStep.System.Web.HttpApplication.IExecutionStep.Execute() +181
   System.Web.HttpApplication.ExecuteStep(IExecutionStep step, Boolean& completedSynchronously) +75
```

The problem is that the event source specified in the `initializeData` attribute (`"Fabrikam  Site"`) does not exist and the service account doesn't have permission  to create a new event source. You might think that you could just omit the `initializeData` attribute -- and use whatever  default is specified in the **EventLogTraceListener** -- but, alas,  that only leads to a `NullReferenceException` and a slightly different  call stack.

Consequently, we need to first create the event source that we want to use.

Unfortunately, I'm not aware of any way to do this using OOTB functionality (e.g.  from the Event Viewer console). Fortunately, it takes only a tiny bit of custom  code to create the event source (assuming you have permissions to do so):

```
private static string AddEventLogSource(
            string source)
        {
            Debug.Assert(string.IsNullOrEmpty(source) == false);

            bool sourceExists = EventLog.SourceExists(source);
            if (sourceExists == false)
            {
                EventLog.CreateEventSource(source, "Application");
            }

            return "Operation completed successfully.";
        }
```

However, even after creating the event source, you still can't use the **EventLogTraceListener** on a SharePoint site configured for Forms-Based  Authentication and anonymous access. Attempting to do so results in the following  error:

```
[Win32Exception (0x80004005): The handle is invalid]
   System.Diagnostics.EventLog.InternalWriteEvent(UInt32 eventID, UInt16 category, EventLogEntryType type, String[] strings, Byte[] rawData, String currentMachineName) +517
   System.Diagnostics.EventLog.WriteEvent(EventInstance instance, Byte[] data, Object[] values) +313
   System.Diagnostics.EventLog.WriteEvent(EventInstance instance, Object[] values) +10
   System.Diagnostics.EventLogTraceListener.TraceEvent(TraceEventCache eventCache, String source, TraceEventType severity, Int32 id, String message) +107
   System.Diagnostics.TraceSource.TraceEvent(TraceEventType eventType, Int32 id, String message) +195
   Fabrikam.Demo.CoreServices.Logging.Logger.Log(TraceEventType eventType, String message) +211
   Fabrikam.Demo.CoreServices.Logging.Logger.LogError(Exception ex, Uri requestUrl) +773
   Fabrikam.Demo.Publishing.Layouts.MasterPages.FabrikamMinimal.Page_Error(Object sender, EventArgs e) +86
   System.Web.UI.TemplateControl.OnError(EventArgs e) +8690682
   System.Web.UI.Page.HandleError(Exception e) +84
   System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint) +6776
   System.Web.UI.Page.ProcessRequest(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint) +242
   System.Web.UI.Page.ProcessRequest() +80
   System.Web.UI.Page.ProcessRequestWithNoAssert(HttpContext context) +21
   System.Web.UI.Page.ProcessRequest(HttpContext context) +49
   ASP.WELCOMESPLASH_ASPX__533642652.ProcessRequest(HttpContext context) +4
   Microsoft.SharePoint.Publishing.TemplateRedirectionPage.ProcessRequest(HttpContext context) +184
   System.Web.CallHandlerExecutionStep.System.Web.HttpApplication.IExecutionStep.Execute() +181
   System.Web.HttpApplication.ExecuteStep(IExecutionStep step, Boolean& completedSynchronously) +75
```

To workaround this problem, I created a custom trace listener by essentially  snarfing the code for the **EventLogTraceListener** class and wrapping  the calls to write events with `SPSecurity.RunWithElevatedPrivileges`:

```
using System;
using System.Diagnostics;
using System.Globalization;
using System.Security.Permissions;
using System.Runtime.InteropServices;
using System.Text;

using Microsoft.SharePoint;

namespace Fabrikam.Demo.CoreServices.SharePoint
{
    /// <summary>
    /// Provides a simple listener that directs tracing or debugging output
    /// from a SharePoint Web application to an Event Log.
    /// </summary>
    /// <remarks>Unlike the
    /// <see cref="System.Diagnostics.EventLogTraceListener"/>, this class
    /// writes events using SPSecurity.RunWithElevatedPrivileges (in order to
    /// avoid "handle is invalid" errors when using Forms-Based Authentication
    /// and anonymous users).</remarks>
    [HostProtection(SecurityAction.LinkDemand, Synchronization = true)]
    public sealed class SharePointEventLogTraceListener : TraceListener
    {
        #region Fields

        private bool disposed;
        private EventLog eventLog;
        private bool nameSet;

        #endregion

        #region Constructors

        /// <summary>
        /// Initializes a new instance of the
        /// <see cref="SharePointEventLogTraceListener" /> class without a trace
        /// listener.
        /// </summary>
        public SharePointEventLogTraceListener()
        {
        }

        /// <summary>
        /// Initializes a new instance of the
        /// <see cref="SharePointEventLogTraceListener" /> class using the
        /// specified event log.
        /// </summary>
        /// <param name="eventLog">An
        /// <see cref="System.Diagnostics.EventLog"/> that specifies the event
        /// log to write to.</param>
        public SharePointEventLogTraceListener(
            EventLog eventLog)
            : base((eventLog != null) ? eventLog.Source : string.Empty)
        {
            this.eventLog = eventLog;
        }

        /// <summary>
        /// Initializes a new instance of the
        /// <see cref="SharePointEventLogTraceListener" /> class using the
        /// specified source.
        /// </summary>
        /// <param name="source">The name of an existing event log source.</param>
        public SharePointEventLogTraceListener(
            string source)
        {
            this.eventLog = new EventLog();
            this.eventLog.Source = source;
        }

        #endregion

        #region IDisposable implementation

        /// <summary>
        /// Attempts to free resources and perform other cleanup operations
        /// before the object is reclaimed by garbage collection.
        /// </summary>
        /// <remarks>
        /// Assuming all instances of the
        /// <see cref="SharePointEventLogTraceListener"/>
        /// class are properly disposed, the finalizer should never be invoked.
        /// </remarks>
        ~SharePointEventLogTraceListener()
        {
            Debug.Fail(
                "SharePointEventLogTraceListener was not properly disposed.");

            Dispose(false);
        }

        /// <summary>
        /// Releases all resources associated with an instance of the class.
        /// </summary>
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Design",
            "CA1063:ImplementIDisposableCorrectly")]
        public new void Dispose()
        {
            Dispose(true);
            GC.SuppressFinalize(this);
        }

        /// <summary>
        /// Releases resources associated with an instance of the class.
        /// </summary>
        /// <param name="disposing"><c>true</c> if managed resources should be
        /// disposed; <c>false</c> if only unmanaged resources should be disposed.
        /// </param>
        protected override void Dispose(
            bool disposing)
        {
            base.Dispose(disposing);

            if (disposed == false)
            {
                if (disposing)
                {
                    // Release all managed resources here
                    if (this.eventLog != null)
                    {
                        this.eventLog.Dispose();
                        this.eventLog = null;
                    }
                }

                // Release all unmanaged resources here
            }

            disposed = true;
        }

        #endregion

        #region Methods

        /// <summary>
        /// Closes the event log so that it no longer receives tracing or
        /// debugging output.
        /// </summary>
        public override void Close()
        {
            if (this.eventLog != null)
            {
                this.eventLog.Close();
            }
        }

        private static EventInstance CreateEventInstance(
            TraceEventType severity,
            int id)
        {
            if (id > 0xffff)
            {
                id = 0xffff;
            }

            if (id < 0)
            {
                id = 0;
            }

            EventInstance instance = new EventInstance((long)id, 0);
            if (severity == TraceEventType.Error
                || severity == TraceEventType.Critical)
            {
                instance.EntryType = EventLogEntryType.Error;
                return instance;
            }

            if (severity == TraceEventType.Warning)
            {
                instance.EntryType = EventLogEntryType.Warning;
                return instance;
            }

            instance.EntryType = EventLogEntryType.Information;
            return instance;
        }

        // HACK: Mimic the various internal overloads of
        // TraceFilter.ShouldTrace to minimize code changes from
        // System.Diagnostics.EventLogTraceListener
        private bool ShouldTrace(
            TraceEventCache cache,
            string source,
            TraceEventType eventType,
            int id,
            string formatOrMessage)
        {
            return this.ShouldTrace(
                cache,
                source,
                eventType,
                id,
                formatOrMessage,
                null,
                null,
                null);
        }

        private bool ShouldTrace(
            TraceEventCache cache,
            string source,
            TraceEventType eventType,
            int id,
            string formatOrMessage,
            object[] args)
        {
            return this.ShouldTrace(
                cache,
                source,
                eventType,
                id,
                formatOrMessage,
                args,
                null,
                null);
        }

        private bool ShouldTrace(
            TraceEventCache cache,
            string source,
            TraceEventType eventType,
            int id,
            string formatOrMessage,
            object[] args,
            object data1)
        {
            return this.ShouldTrace(
                cache,
                source,
                eventType,
                id,
                formatOrMessage,
                args,
                data1,
                null);
        }

        private bool ShouldTrace(
            TraceEventCache cache,
            string source,
            TraceEventType eventType,
            int id,
            string formatOrMessage,
            object[] args,
            object data1,
            object[] data)
        {
            if (base.Filter == null)
            {
                return true;
            }

            return base.Filter.ShouldTrace(
                cache,
                source,
                eventType,
                id,
                formatOrMessage,
                args,
                data1,
                data);
        }

        /// <summary>
        /// Writes trace information, an array of data objects and event
        /// information to the event log.
        /// </summary>
        /// <param name="eventCache">A
        /// <see cref="System.Diagnostics.TraceEventCache"/> object that
        /// contains the current process ID, thread ID, and stack trace
        /// information.</param>
        /// <param name="source">A name used to identify the output, typically
        /// the name of the application that generated the trace event.</param>
        /// <param name="eventType">One of the
        /// <see cref="System.Diagnostics.TraceEventType"/> values specifying
        /// the type of event that has caused the trace.</param>
        /// <param name="id">A numeric identifier for the event. The combination
        /// of <paramref name="source"/> and <paramref name="id"/> uniquely
        /// identifies an event.</param>
        /// <param name="data">An array of data objects.</param>
        [ComVisible(false)]
        public override void TraceData(
            TraceEventCache eventCache,
            string source,
            TraceEventType eventType,
            int id,
            params object[] data)
        {
            bool shouldTrace = this.ShouldTrace(
                eventCache,
                source,
                eventType,
                id,
                null,
                null,
                null,
                data);

            if (shouldTrace == true)
            {
                EventInstance instance = CreateEventInstance(eventType, id);
                StringBuilder builder = new StringBuilder();
                if (data != null)
                {
                    for (int i = 0; i < data.Length; i++)
                    {
                        if (i != 0)
                        {
                            builder.Append(", ");
                        }
                        if (data[i] != null)
                        {
                            builder.Append(data[i].ToString());
                        }
                    }
                }

                SPSecurity.RunWithElevatedPrivileges(
                delegate()
                {
                    this.eventLog.WriteEvent(
                        instance,
                        new object[] { builder.ToString() });
                });
            }
        }

        /// <summary>
        /// Writes trace information, a data object and event information to the
        /// event log.
        /// </summary>
        /// <param name="eventCache">A
        /// <see cref="System.Diagnostics.TraceEventCache"/> object that
        /// contains the current process ID, thread ID, and stack trace
        /// information.</param>
        /// <param name="source">A name used to identify the output, typically
        /// the name of the application that generated the trace event.</param>
        /// <param name="eventType">One of the
        /// <see cref="System.Diagnostics.TraceEventType"/> values specifying
        /// the type of event that has caused the trace.</param>
        /// <param name="id">A numeric identifier for the event. The combination
        /// of <paramref name="source"/> and <paramref name="id"/> uniquely
        /// identifies an event.</param>
        /// <param name="data">A data object to write to the output file or
        /// stream.</param>
        [ComVisible(false)]
        public override void TraceData(
            TraceEventCache eventCache,
            string source,
            TraceEventType eventType,
            int id,
            object data)
        {
            bool shouldTrace = this.ShouldTrace(
                eventCache,
                source,
                eventType,
                id,
                null,
                null,
                data);

            if (shouldTrace == true)
            {
                EventInstance instance = CreateEventInstance(eventType, id);

                SPSecurity.RunWithElevatedPrivileges(
                delegate()
                {
                    this.eventLog.WriteEvent(instance, new object[] { data });
                });
            }
        }

        /// <summary>
        /// Writes trace information, a message and event information to the
        /// event log.
        /// </summary>
        /// <param name="eventCache">A
        /// <see cref="System.Diagnostics.TraceEventCache"/> object that
        /// contains the current process ID, thread ID, and stack trace
        /// information.</param>
        /// <param name="source">A name used to identify the output, typically
        /// the name of the application that generated the trace event.</param>
        /// <param name="eventType">One of the
        /// <see cref="System.Diagnostics.TraceEventType"/> values specifying
        /// the type of event that has caused the trace.</param>
        /// <param name="id">A numeric identifier for the event. The combination
        /// of <paramref name="source"/> and <paramref name="id"/> uniquely
        /// identifies an event.</param>
        /// <param name="message">The trace message.</param>
        [ComVisible(false)]
        public override void TraceEvent(
            TraceEventCache eventCache,
            string source,
            TraceEventType eventType,
            int id,
            string message)
        {
            bool shouldTrace = this.ShouldTrace(
                eventCache,
                source,
                eventType,
                id,
                message);

            if (shouldTrace == true)
            {
                EventInstance instance = CreateEventInstance(eventType, id);

                SPSecurity.RunWithElevatedPrivileges(
                delegate()
                {
                    this.eventLog.WriteEvent(instance, new object[] { message });
                });
            }
        }

        /// <summary>
        /// Writes trace information, a formatted array of objects and event
        /// information to the event log.
        /// </summary>
        /// <param name="eventCache">A
        /// <see cref="System.Diagnostics.TraceEventCache"/> object that
        /// contains the current process ID, thread ID, and stack trace
        /// information.</param>
        /// <param name="source">A name used to identify the output, typically
        /// the name of the application that generated the trace event.</param>
        /// <param name="eventType">One of the
        /// <see cref="System.Diagnostics.TraceEventType"/> values specifying
        /// the type of event that has caused the trace.</param>
        /// <param name="id">A numeric identifier for the event. The combination
        /// of <paramref name="source"/> and <paramref name="id"/> uniquely
        /// identifies an event.</param>
        /// <param name="format">A format string that contains zero or more
        /// format items that correspond to objects in the
        /// <paramref name="args"/> array.</param>
        /// <param name="args">An <c>object</c> array containing zero or more
        /// objects to format.</param>
        [ComVisible(false)]
        public override void TraceEvent(
            TraceEventCache eventCache,
            string source,
            TraceEventType eventType,
            int id,
            string format,
            params object[] args)
        {
            bool shouldTrace = this.ShouldTrace(
                eventCache,
                source,
                eventType,
                id,
                format,
                args);

            if (shouldTrace == true)
            {
                EventInstance instance = CreateEventInstance(eventType, id);
                if (args == null)
                {
                    SPSecurity.RunWithElevatedPrivileges(
                    delegate()
                    {
                        this.eventLog.WriteEvent(
                            instance,
                            new object[] { format });
                    });
                }
                else if (string.IsNullOrEmpty(format))
                {
                    string[] values = new string[args.Length];
                    for (int i = 0; i < args.Length; i++)
                    {
                        values[i] = args[i].ToString();
                    }

                    SPSecurity.RunWithElevatedPrivileges(
                    delegate()
                    {
                        this.eventLog.WriteEvent(instance, values);
                    });
                }
                else
                {
                    SPSecurity.RunWithElevatedPrivileges(
                    delegate()
                    {
                        this.eventLog.WriteEvent(
                            instance,
                            new object[]
                            {
                                string.Format(
                                    CultureInfo.InvariantCulture,
                                    format,
                                    args)
                            });
                    });
                }
            }
        }

        /// <summary>
        /// Writes a message to the event log for this instance.
        /// </summary>
        /// <param name="message">A message to write.</param>
        public override void Write(
            string message)
        {
            if (this.eventLog != null)
            {
                SPSecurity.RunWithElevatedPrivileges(
                delegate()
                {
                    this.eventLog.WriteEntry(message);
                });
            }
        }

        /// <summary>
        /// Writes a message to the event log for this instance.
        /// </summary>
        /// <param name="message">A message to write.</param>
        public override void WriteLine(
            string message)
        {
            this.Write(message);
        }

        #endregion

        #region Properties

        /// <summary>
        /// Gets or sets the event log to write to.
        /// </summary>
        public EventLog EventLog
        {
            get
            {
                return this.eventLog;
            }
            set
            {
                this.eventLog = value;
            }
        }

        /// <summary>
        /// Gets or sets the name of this
        /// <see cref="SharePointEventLogTraceListener" />.
        /// </summary>
        public override string Name
        {
            get
            {
                if (this.nameSet == false
                    && this.eventLog != null)
                {
                    base.Name = this.eventLog.Source;
                    this.nameSet = true;
                }

                return base.Name;
            }
            set
            {
                this.nameSet = true;
                base.Name = value;
            }
        }

        #endregion
    }
}
```

Now, we can simply replace **System.Diagnostics.EventLogTraceListener**  with the custom **Fabrikam.Demo.CoreServices.SharePoint.SharePointEventLogTraceListener**  in the Web.config file:

```
<system.diagnostics>
    <sources>
      <source name="defaultTraceSource" switchName="allTraceLevel">
        <listeners>
          <add name="eventLogTraceListener" />
        </listeners>
      </source>
    </sources>
    <sharedListeners>
      <add type="Fabrikam.Demo.CoreServices.SharePoint.SharePointEventLogTraceListener,
          Fabrikam.Demo.CoreServices.SharePoint, Version=1.0.0.0,
          Culture=neutral, PublicKeyToken=786f58ca4a6e3f60"
        name="eventLogTraceListener"
        initializeData="Fabrikam Site">
        <filter type="System.Diagnostics.EventTypeFilter"
          initializeData="Information" />
      </add>
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

...and, voilà! Everything just works!

Woohoo!!

Okay, maybe I should omit the word "just" from that statement, but oh well...

The point is, when we now encounter an unhandled exception on any page in the  site that uses our custom master page, instead of showing the OOTB SharePoint error  page, we instead show a nicely-branded custom error page. Just as important, the  details of the exception are logged to the **Application** event log.

Here's the event that I just captured from my sample Web Part:

```
Log Name:      Application
Source:        Fabrikam Site
Date:          3/20/2010 5:12:39 PM
Event ID:      0
Task Category: None
Level:         Error
Keywords:      Classic
User:          N/A
Computer:      foobar2.corp.technologytoolbox.com
Description:
An error occurred during the execution of the web request. Please review the stack trace for more information about the error and where it originated in the code.

Request URL: http://www-local.fabrikam.com/Pages/default.aspx

Exception Details: System.DivideByZeroException: Attempted to divide by zero.

Stack Trace:

[DivideByZeroException: Attempted to divide by zero.]
   at Fabrikam.Demo.WebParts.UnhandledExceptionWebPart.DoSomethingBad()

[InvalidOperationException: Something bad happened.]
   at Fabrikam.Demo.WebParts.UnhandledExceptionWebPart.DoSomethingBad()
   at Fabrikam.Demo.WebParts.UnhandledExceptionWebPart.throwExceptionButton_Click(Object sender, EventArgs e)
   at System.Web.UI.WebControls.LinkButton.OnClick(EventArgs e)
   at System.Web.UI.WebControls.LinkButton.RaisePostBackEvent(String eventArgument)
   at System.Web.UI.WebControls.LinkButton.System.Web.UI.IPostBackEventHandler.RaisePostBackEvent(String eventArgument)
   at System.Web.UI.Page.RaisePostBackEvent(IPostBackEventHandler sourceControl, String eventArgument)
   at System.Web.UI.Page.RaisePostBackEvent(NameValueCollection postData)
   at System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)
```

I don't know about you, but I think that's pretty darn cool!

Before I wrap things up for this post, I want to point out that I'm not very  fond of manually modifying configuration files. As I've stated before, this is prone  to human error and quickly becomes tedious when you have to do it over and over  again for different environments or whenever you decide to nuke your LOCAL or DEV  SharePoint Web application and recreate it (which is something I tend to do rather  frequently).

Consequently, I use custom StsAdm.exe commands to add the event source and enable  logging to the event log via the`<system.diagnostics>`  configuration elements shown above.

After deploying the custom StsAdm.exe commands (Fabrikam.Demo.StsAdm.Command.wsp),  the following command must be run once on each Web server in each SharePoint environment  (e.g. LOCAL, DEV, TEST, and PROD):

```
stsadm -o fabrikam-addeventlogsource -source "Fabrikam Site"
```

The following command, on the other hand, only needs to be run once per environment  after creating the Web application (substituting the appropriate URL as necessary)  since the SharePoint **SPWebConfigModification** class handles the  grunt work of modifying the Web.config files for each zone and on each front-end  Web server in the farm:

```
stsadm -o fabrikam-enablelogging -url http://fabrikam
```

> **Note**
> 
> There's a known bug with the **SPWebConfigModification** class
> not removing modifications from any Web.config file except the one for the
> **Default** zone. In other words, running the following command
> will not remove the trace listener configuration from the Web.config file
> for the **Internet** zone:
> 
> ```
> stsadm -o fabrikam-disablelogging -url http://fabrikam
> ```

Here is the class that implements the custom StsAdm.exe commands:

```
using System;
using System.Collections.Specialized;
using System.Diagnostics;
using System.Globalization;
using System.Security.Permissions;

using Microsoft.SharePoint.Administration;
using Microsoft.SharePoint.Security;
using Microsoft.SharePoint.StsAdmin;

using Fabrikam.Demo.CoreServices.SharePoint;

namespace Fabrikam.Demo.StsAdm.Commands
{
    /// <summary>
    /// Adds new command-line operations and new functionality to the
    /// STSADM.EXE utility for enabling and configuring diagnostics.
    /// </summary>
    public class DiagnosticsCommands : ISPStsadmCommand
    {
        private const string loggingWebConfigModificationOwner =
"Fabrikam.Demo.StsAdm.Commands.DiagnosticsCommands";

        #region Help text

        private const string ADD_EVENT_LOG_SOURCE_HELP =
            "Ensures the specified event source exists on the local computer.\r\n"
            + "\r\n       -source <The source name for logging to the Event Log.>";

        private const string ENABLE_LOGGING_HELP =
            "Adds the necessary configuration changes to enable logging on a web application.\r\n"
            + "\r\n       -url <url of the Web application to enable logging on>";

        private const string DISABLE_LOGGING_HELP =
            "Removes the event logging configuration from a web application.\r\n"
            + "\r\n       -url <url of the Web application to remove logging from>";

        #endregion

        #region ISPStsadmCommand Members

        /// <summary>
        /// Gets a remark about the syntax of the specified custom operation.
        /// </summary>
        /// <param name="command">The operation about which help is sought.
        /// </param>
        /// <returns>A <see cref="String"/> that represents a description of
        /// the syntax of the operation <paramref name="command"/>.</returns>
        [SharePointPermission(SecurityAction.LinkDemand, ObjectModel = true)]
        public string GetHelpMessage(
            string command)
        {
            if (command == null)
            {
                throw new ArgumentNullException("command");
            }

            command = command.Trim();
            if (string.IsNullOrEmpty(command) == true)
            {
                throw new ArgumentException(
                    "The command must be specified.",
                    "command");
            }

            switch (command)
            {
                case "fabrikam-addeventlogsource":
                    return ADD_EVENT_LOG_SOURCE_HELP;
                case "fabrikam-enablelogging":
                    return ENABLE_LOGGING_HELP;
                case "fabrikam-disablelogging":
                    return DISABLE_LOGGING_HELP;
                default:
                    string message = string.Format(
                        CultureInfo.CurrentUICulture,
                        "The specified command ({0}) is not valid.",
                        command);

                    throw new ArgumentException(
                        message,
                        "command");
            }
        }

        /// <summary>
        /// Executes the specified operation.
        /// </summary>
        /// <param name="command">The name of the custom operation.</param>
        /// <param name="keyValues">The parameters, if any, that are added to
        /// the command line.</param>
        /// <param name="output">An output string, if needed.</param>
        /// <returns>An <see cref="Int32" /> that can be used to signal the
        /// result of the operation. For proper operation with STSADM, use the
        /// following rules when implementing. Return 0 for success. Return
        /// <see cref="Microsoft.SharePoint.StsAdmin.ErrorCodes.GeneralError" />
        /// (-1) for any error other than syntax. Return
        /// <see cref="Microsoft.SharePoint.StsAdmin.ErrorCodes.SyntaxError" />
        /// (-2) for a syntax error. When 0 is returned, STSADM streams
        /// <paramref name="output"/>, if it is not <c>null</c>, to the console.
        /// When <c>SyntaxError</c> is returned, STSADM calls
        /// <see cref="GetHelpMessage" /> and its return value is streamed to
        /// the console, and it streams output, if it is not <c>null</c>, to
        /// stderr (standard error). To obtain the content of stderr in managed
        /// code, use <see cref="System.Console.Error" />. When any other value
        /// is returned, STSADM streams <paramref name="output"/>, if it is not
        /// <c>null</c>, to stderr.</returns>
        [SharePointPermission(SecurityAction.LinkDemand, ObjectModel = true)]
        public int Run(
            string command,
            StringDictionary keyValues,
            out string output)
        {
            if (command == null)
            {
                throw new ArgumentNullException("command");
            }

            command = command.Trim();
            if (string.IsNullOrEmpty(command) == true)
            {
                throw new ArgumentException(
                    "The command must be specified.",
                    "command");
            }

            if (keyValues == null)
            {
                throw new ArgumentNullException("keyValues");
            }

            command = command.ToUpperInvariant();
            switch (command)
            {
                case "FABRIKAM-ADDEVENTLOGSOURCE":
                    return AddEventLogSource(keyValues, out output);

                case "FABRIKAM-ENABLELOGGING":
                    return EnableLogging(keyValues, out output);

                case "FABRIKAM-DISABLELOGGING":
                    return DisableLogging(keyValues, out output);

                default:
                    string message = string.Format(
                        CultureInfo.CurrentUICulture,
                        "The specified command ({0}) is not valid.",
                        command);

                    throw new ArgumentException(
                        message,
                        "command");
            }
        }

        #endregion

        #region AddEventLogSource

        private static int AddEventLogSource(
            StringDictionary keyValues,
            out string output)
        {
            try
            {
                string source = CommandHelper.GetKeyValue(
                    keyValues,
                    "source",
                    true);

                output = AddEventLogSource(
                    source);
            }
            catch (ArgumentException ex)
            {
                output = ex.Message;
                return (int)ErrorCodes.SyntaxError;
            }

            return 0;
        }

        private static string AddEventLogSource(
            string source)
        {
            Debug.Assert(string.IsNullOrEmpty(source) == false);

            bool sourceExists = EventLog.SourceExists(source);
            if (sourceExists == false)
            {
                EventLog.CreateEventSource(source, "Application");
            }

            return "Operation completed successfully.";
        }

        #endregion

        #region EnableLogging

        private static int EnableLogging(
            StringDictionary keyValues,
            out string output)
        {
            try
            {
                string webAppUrl = CommandHelper.GetKeyValue(keyValues, "url", true);

                SPWebApplication webApp = SPWebApplication.Lookup(
                    new Uri(webAppUrl));

                output = EnableLogging(
                    webApp);
            }
            catch (ArgumentException ex)
            {
                output = ex.Message;
                return (int)ErrorCodes.SyntaxError;
            }

            return 0;
        }

        private static string EnableLogging(
            SPWebApplication webApp)
        {
            Debug.Assert(webApp != null);

            AddLoggingWebConfigModifications(webApp);
            webApp.Update();

            SharePointWebConfigHelper.ApplyWebConfigModifications(webApp);

            return "Operation completed successfully.";
        }

        #endregion

        #region DisableLogging

        private static int DisableLogging(
            StringDictionary keyValues,
            out string output)
        {
            try
            {
                string webAppUrl = CommandHelper.GetKeyValue(keyValues, "url", true);

                SPWebApplication webApp = SPWebApplication.Lookup(
                    new Uri(webAppUrl));

                output = DisableLogging(
                    webApp);
            }
            catch (ArgumentException ex)
            {
                output = ex.Message;
                return (int)ErrorCodes.SyntaxError;
            }

            return 0;
        }

        private static string DisableLogging(
            SPWebApplication webApp)
        {
            Debug.Assert(webApp != null);

            SharePointWebConfigHelper.RemoveWebConfigModifications(
                webApp,
                loggingWebConfigModificationOwner);

            return "Operation completed successfully.";
        }

        #endregion
        
        private static void AddLoggingWebConfigModifications(
            SPWebApplication webApp)
        {
            Debug.Assert(webApp != null);

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                loggingWebConfigModificationOwner,
                "trace",
                "configuration/system.web",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
"<trace"
    + " enabled='true'"
    + " localOnly='true'"
    + " pageOutput='false'"
    + " requestLimit='50' />");

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                loggingWebConfigModificationOwner,
                "system.diagnostics",
                "configuration",
                SPWebConfigModification.SPWebConfigModificationType.EnsureSection,
                null);

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                loggingWebConfigModificationOwner,
                "sources",
                "configuration/system.diagnostics",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
"<sources>"
      + "<source name='defaultTraceSource' switchName='allTraceLevel'>"
        + "<listeners>"
            + "<add name='webPageTraceListener' />"
            + "<add name='eventLogTraceListener' />"
        + "</listeners>"
    + "</source>"
+ "</sources>");

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                loggingWebConfigModificationOwner,
                "sharedListeners",
                "configuration/system.diagnostics",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
"<sharedListeners>"
    + "<add type='System.Web.WebPageTraceListener,"
            + " System.Web, Version=2.0.0.0,"
            + " Culture=neutral,"
            + " PublicKeyToken=b03f5f7f11d50a3a'"
        + " name='webPageTraceListener'"
        + " traceOutputOptions='None' />"
    + "<add type='Fabrikam.Demo.CoreServices.SharePoint.SharePointEventLogTraceListener,"
            + " Fabrikam.Demo.CoreServices.SharePoint,"
            + " Version=1.0.0.0,"
            + " Culture=neutral,"
            + " PublicKeyToken=786f58ca4a6e3f60'"
        + " name='eventLogTraceListener'"
        + " initializeData='Fabrikam Site'>"
        + "<filter type='System.Diagnostics.EventTypeFilter'"
            + " initializeData='Information' />"
    + "</add>"
+ "</sharedListeners>");

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                loggingWebConfigModificationOwner,
                "switches",
                "configuration/system.diagnostics",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
"<switches>"
    + "<add name='allTraceLevel' value='All' />"
    + "<add name='verboseTraceLevel' value='Verbose' />"
    + "<add name='infoTraceLevel' value='Info' />"
    + "<add name='warningTraceLevel' value='Warning' />"
    + "<add name='errorTraceLevel' value='Error' />"
    + "<add name='offTraceLevel' value='Off' />"
+ "</switches>");

        }
    }
}
```

I hope you find this post valuable when creating Internet sites with MOSS 2007.

