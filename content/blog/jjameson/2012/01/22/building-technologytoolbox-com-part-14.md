---
title: "Best practices for error handling in ASP.NET Web applications (a.k.a. Building TechnologyToolbox.com, part 14)"
date: 2012-01-22T03:15:57-07:00
excerpt: "So, you've created a custom error page, enabled the &lt;customErrors&gt; element in the Web.config file, and considered it done -- but are you sure that all the bases are covered?"
aliases: ["/blog/jjameson/archive/2012/01/21/building-technologytoolbox-com-part-14.aspx"]
draft: true
categories: ["Development", "My System"]
tags: ["Web Development"]
---

Software is never perfect. Errors *will* occur in your code. Don't
fret, they occur in my code, too. There are nasty little errors in everyone's
code -- idly biding their time until they can spring out and aggravate your
users.

Some errors might be expected, such as HTTP 404 errors when someone mistypes
a URL -- or when hackers maliciously try to find vulnerabilities in your application.
Other errors might be completely unexpected (e.g. "What do you mean
[the database transaction log is full](/blog/jjameson/2007/05/06/no-error-displayed-when-database-update-fails)? Isn't someone supposed to be monitoring
that?").

Many errors are avoidable, but you will inevitably encounter some situations
where the best thing your site can do is cough up a decent "[mea
maxima culpa](http://en.wikipedia.org/wiki/Mea_culpa)" message and try to avoid showing users the infamous Yellow
Page of Death.

Last year, I blogged my recommendations for
[error handling in SharePoint applications](/blog/jjameson/2010/03/20/error-handling-in-moss-2007-applications), but what if you aren't using
SharePoint? What if you working in plain ol' ASP.NET-land?

In that case, you might already have specified a custom error page in the
Web.config file and considered it done -- but are you sure that all the bases
are covered?

Let's examine a few scenarios...

### Unhandled Exceptions

Unless you are writing sample code to demonstrate something as simple as
basic calculator functions, you can't possibly expect your code to gracefully
handle the multitude of exceptions that might occur at runtime. Therefore, the
fewer `try`/`catch` blocks you add to your code, the better.

Instead you should only catch an exception when you are absolutely sure you
can do something useful. If you have `catch` blocks in your code
that do nothing more than log the exception and then re-throw it up the call
stack, then...well, I'll just say it, your code *blows*.

Okay, maybe that's a little harsh, but you wouldn't believe how often I've
seen this in the past. Even worse, though, are `catch` blocks that
"swallow" the exceptions. If you think there's even a remote chance you might
have these in your solution, then you should stop reading this post *immediately*
and instead go do a search in Visual Studio for the word "catch" and scan through
the results one-by-one.

I'm certainly not saying that you should never use `try`/`catch`
blocks in your code. There are times when these are very useful. For example,
when developing a Web Part (regardless of whether it be for ASP.NET or SharePoint),
should an unexpected exception in the Web Part cause the entire page to "blow
chunks" or should an error message prominently appear in place of the Web Part
instead? I typically prefer the latter behavior, because it makes the troubleshooting
go much faster and is arguably less aggravating for users (depending, of course,
on the specific circumstances).

So if your code makes minimal use of `try`/`catch`
blocks and, as I've already said, errors will inevitably occur in your application,
then how should you "handle" these exceptions?

The first step is to use something like ELMAH. If you are more familiar with
SharePoint development and you haven't heard of ELMAH, just think of it as an
error handling stack for ASP.NET that is similar to what you get out-of-the-box
with SharePoint.

If, like me, you've used ELMAH in the past, then you know it takes a little
effort to integrate it into your solution. Or, rather it used to...

Thanks to [NuGet](http://nuget.org/), you can now add ELMAH to
your solution in a matter of seconds (literally). All you need to do is run
the following command in the
[Package Manager Console](http://docs.nuget.org/docs/start-here/using-the-package-manager-console):

{{< console-block-start >}}

PM&gt; {{< kbd "Install-Package elmah" >}}

{{< console-block-end >}}

Refer to the following resource for more information on the ELMAH NuGet package:

{{< reference title="NuGet Gallery - ELMAH" linkHref="http://nuget.org/packages/elmah" >}}

Note that there are a number of different "flavors" of ELMAH NuGet packages
available (depending on how your want to log errors). For example, refer to
one of Scott Hanselman's posts for more detail on using
[ELMAH with SQL Server Compact Edition](http://www.hanselman.com/blog/NuGetPackageOfTheWeek7ELMAHErrorLoggingModulesAndHandlersWithSQLServerCompact.aspx).

With ELMAH in place, most of the "heavy lifting" has already been performed
for you. To see what happens when an unhandled exception occurs in your ASP.NET
application, you could add an "error simulator" to your site (e.g. SimulateError.ashx):

```
using System;
using System.Web;

namespace TechnologyToolbox.Caelum.Website.Errors
{
    /// <summary>
    /// Simulates an unhandled exception on the site (useful for demonstrating
    /// or troubleshooting the error handling functionality on the site).
    /// </summary>
    public class SimulateErrorHandler : IHttpHandler
    {
        public void ProcessRequest(
            HttpContext context)
        {
            throw new InvalidOperationException(
                "Imagine something interesting here...");
        }

        public bool IsReusable
        {
            get
            {
                return false;
            }
        }
    }
}
```

Browsing to the "error simulator" triggers an unhandled exception and subsequently
the ASP.NET Yellow Page of Death. However, something interesting happened in
between. ELMAH logged the details of the error. Depending on how you configure
ELMAH to log messages, you can use a variety of methods to review and investigate
errors.

If the volume of traffic on your site is relatively low, you might consider
configuring ELMAH to send you an email whenever something bad happens:

```
<elmah>
    <errorMail
      from="no-reply@technologytoolbox.com"
      to="web-support@technologytoolbox.com"
      priority="High"
      smtpPort="25"
      smtpServer="smtp.technologytoolbox.com" />
  </elmah>
```

You might prefer some other notification mechanism, such as
[configuring an alert in Operations Manager whenever an error is written to the
Event Log](/blog/jjameson/2011/03/18/operations-manager-alerts-for-event-log-errors). The point is, you either need to use some form of "push notification"
whenever an error occurs (preferably) or else be very diligent about periodically
reviewing the logs to check for errors (which likely won't happen as time goes
on).

With the logging functionality out of the way, let's turn our attention to
improving the user experience by adding a custom error page.

### Custom Error Page

If you happen to encounter an unexpected error on the Technology Toolbox
site, then you'll see something similar to the screenshot below.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Error-Generic.png"
alt="Custom error page (Generic.aspx)"
height="295"
width="600"
title="Figure 1: Custom error page (Generic.aspx)" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Error-Generic.png)

You've probably created a similar error page before and "wired it up" in
the Web.config file using something like the following:

```
<system.web>
    ...
    <customErrors defaultRedirect="~/Errors/Generic.aspx" mode="On" />
    ...
  </system.web>
```

Note that if you don't specify the `redirectMode`
attribute, it defaults to **[ResponseRedirect](http://msdn.microsoft.com/en-us/library/system.web.configuration.customerrorsredirectmode.aspx)**, which causes the URL to change to something like
the following:

> http://.../Errors/Generic.aspx?aspxerrorpath=...

Consequently if users attempt to refresh the page, they are simply requesting
the error page again (in this case, "Generic.aspx") -- not the original URL
that caused the error. In other words, if the original problem was caused by
some transient condition, with this configuration users could keep clicking
to refresh the page all day long and nothing would ever change. That's certainly
not a good thing -- so you should always specify `redirectMode="ResponseRewrite"`:

```
<customErrors defaultRedirect="~/Errors/Generic.aspx" mode="On"
      redirectMode="ResponseRewrite" />
```

With this configuration, the original request URL is preserved and there's
*at least a chance* that clicking "refresh" will resolve the issue.

However, there's still a big problem...

If your custom error page is simply an ASP.NET page with no code-behind,
then the HTTP response specifies a status code of 200 ("OK"). In other words,
while users will know that something is wrong, search engines crawling your
site will assume the error page content is what you *intended* to serve
for the specified URL.

Consequently, you should add a little code to set the status code to indicate
something went wrong:

```
protected void Page_Load(
               object sender,
               EventArgs e)
        {
            Exception ex = Server.GetLastError();

            HttpException httpEx = ex as HttpException;

            if (httpEx != null)
            {
                Response.StatusCode = httpEx.GetHttpCode();
            }
            else
            {
                Response.StatusCode = 500;
            }
```

Note that this code attempts to preserve the status code of the underlying
**HttpException** (if one occurred). Setting the status code to
indicate an error (e.g. 500) allows search engines to determine the content
in the response should not be added to the search index.

For the Technology Toolbox site, the error page shown in Figure 1 is actually
rendered by two items: **Generic.aspx** and **Error.master**.

The reason for using a different master page is to minimize the potential
for another error to occur during the process of handling the original error.

If this doesn't make sense, consider what could happen if the master page
used by Generic.aspx has a dependency on a database (for example, to render
the global navigation at the top of the page) and for some reason SQL Server
is unavailable. In that scenario, another exception would occur while trying
to render the custom error page. [As an aside, I believe this is precisely why
SharePoint makes you specify a static HTML page in the error handling configuration.]

For the Technology Toolbox site, most of the implementation resides in the
master page (in other words, the content page is simply a "shell"). To understand
why, refer to the next section (HTTP 404 Errors).
The master page provides just enough HTML to indicate that something went wrong,
along with links back to the home page (via the company logo) or to submit additional
information (via the "Contact" form).

#### Generic.aspx

```
<%@ Page Title="Error - Technology Toolbox" Language="C#" AutoEventWireup="true"
  CodeBehind="Generic.aspx.cs" MasterPageFile="~/Errors/Error.master"
  Inherits="TechnologyToolbox.Caelum.Website.Errors.GenericErrorPage" %>
```

#### Generic.aspx.cs

```
using System.Web.UI;

namespace TechnologyToolbox.Caelum.Website.Errors
{
    public partial class GenericErrorPage : Page
    {
    }
}
```

#### Error.master

```
<%@ Master Language="C#" AutoEventWireup="true" CodeBehind="Error.master.cs"
  Inherits="TechnologyToolbox.Caelum.Website.Errors.ErrorMasterPage" %>

<%@ Register TagPrefix="caelum"
  Namespace="TechnologyToolbox.Caelum.Website.Controls"
  Assembly="TechnologyToolbox.Caelum.Website, Version=1.0.0.0, Culture=neutral, PublicKeyToken=f55b5d7768fcda39" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
  <meta content="text/html; charset=utf-8" http-equiv="Content-Type" />
  <title>Error - Technology Toolbox</title>
  <caelum:CssReference runat="server"
    CssFile="~/Themes/Theme-1.0/Theme-1.0.5.min.css"
    DebugCssFile="~/Themes/Theme-1.0/Main.css" />
</head>
<body id="technology-toolbox-com">
  <form runat="server">
  <div id="wrapper">
    <div id="branding">
      <h1>
        <a href="/">
          <img alt="Technology Toolbox"
            src="/Images/TechnologyToolbox-Logo.png" /></a></h1>
    </div>
    <div id="error" class="container_12 clear-fix">
      <div id="contentMain" class="grid_7">
        <h2>
          <asp:ContentPlaceHolder runat="server" ID="PageHeading">
            Error</asp:ContentPlaceHolder>
        </h2>
        <asp:ContentPlaceHolder runat="server" ID="PageContent">
          <p>
            We apologize, but an error occurred and your request could not be
            completed.</p>
          <p>
            This error has been logged. If you have additional information
            regarding what may have caused this error, please
            <a href="/Contact">contact us</a>.</p>
        </asp:ContentPlaceHolder>
      </div>
      <div id="contentSub" class="grid_5">
        <asp:ContentPlaceHolder runat="server" ID="PageImage">
          <img alt="Bug" src="/Images/icon-bug-347x346.jpg"
            width="347" height="346" />
        </asp:ContentPlaceHolder>
      </div>
    </div>
  </div>
  </form>
</body>
</html>
```

#### Error.master.cs

```
using System;
using System.Web;
using System.Web.UI;

namespace TechnologyToolbox.Caelum.Website.Errors
{
    public partial class ErrorMasterPage : MasterPage
    {
        protected void Page_Load(
               object sender,
               EventArgs e)
        {
            Exception ex = Server.GetLastError();

            HttpException httpEx = ex as HttpException;

            if (httpEx != null)
            {
                Response.StatusCode = httpEx.GetHttpCode();
            }
            else
            {
                Response.StatusCode = 500;
            }
        }
    }
}
```

At this point, we've managed to avoid the Yellow Page of Death and instead
display a "branded" error page with a generic error message (all the while logging
the details of the underlying error).

However, what happens if someone mistypes a URL -- or when evil people maliciously
hack the URL looking for vulnerabilities? [Trust me, these people are out there...and
they apparently have nothing better to do with their time than sit there and
try adding things like "/user/CreateUser.aspx" to a URL on your site. (Yes,
this is a real example from my own logs. Shame on you -- whoever you are at
IP 96.31.35.33!)]

### HTTP 404 Errors

Imagine that I send you an email and refer you to my blog, but I mistakenly
type "bog" instead of "blog":

> https://www.technologytoolbox.com/bog/jjameson

If you were to click the link in my email at this point, then you'd see the
the out-of-the-box IIS "404" error page (404.htm) shown below.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_IIS-404.png"
alt="IIS - default 404 page (404.htm)"
height="110"
width="600"
title="Figure 2: IIS - default 404 page (404.htm)" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_IIS-404.png)

You might have expected the custom error page shown in Figure 1 to appear
(based on the `<customErrors>`
element shown above). However, in this scenario, the URL doesn't specify a path
that maps to a managed handler in IIS.

On the other hand, if you try browsing to a non-existent ASP.NET page, such
as:

> https://www.technologytoolbox.com/foobar.aspx

...then you *do* see the error page shown in Figure 1 (and if you
use Firebug or the IE Developer Tools to inspect the response, you would see
that it specifies a status code of 404 -- courtesy of the code above that gets
the status code from the original **HttpException**).

In either case, it would be better to show users a custom 404 error page,
like the one shown below.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Error-404.png"
alt="HTTP 404 error page (404.aspx)"
height="295"
width="600"
title="Figure 3: HTTP 404 error page (404.aspx)" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Error-404.png)

To accomplish this, we simply need to create another ASP.NET page (e.g. 404.aspx)
and configure it to be used in the two scenarios described earlier.

For the first scenario (i.e. when the request specifies a path that does
not correspond to a managed handler), the `<httpErrors>`
element is used:

```
<system.webServer>
    <httpErrors>
      <remove statusCode="404" subStatusCode="-1" />
      <error statusCode="404" prefixLanguageFilePath=""
        path="/Errors/404.aspx" responseMode="ExecuteURL" />
    </httpErrors>
    ...
  </system.webServer>
```

For the second scenario (e.g. when the request specifies a non-existent ASPX
page), the `<customErrors>`
element is used:

```
<system.web>
    ...
    <customErrors defaultRedirect="~/Errors/Generic.aspx" mode="On"
      redirectMode="ResponseRewrite">
      <error statusCode="404" redirect="~/Errors/404.aspx" />
    </customErrors>
    ...
  </system.web>
```

> **Note**
>
> If you'd prefer to configure these settings in IIS Manager (rather than editing the Web.config file directly), under the **IIS** section double-click **Error Pages** (to modify the `<httpErrors>` element) or under the **ASP.NET** section double-click **.NET Error Pages**. (to modify the `<customErrors>` element).

You may have noticed from Figure 3 that I prefer to show the server-relative
URL of the request in the error message. This helps draw the user's attention
to what was specified (in case he or she simply mistyped part of the URL).

#### 404.aspx

```
<%@ Page Title="404 Error - Technology Toolbox" Language="C#"
  AutoEventWireup="true" CodeBehind="404.aspx.cs"
  MasterPageFile="~/Errors/Error.master"
  Inherits="TechnologyToolbox.Caelum.Website.Errors.Error404Page" %>

<asp:Content runat="server" ContentPlaceHolderID="PageHeading">
  HTTP 404 (That's an error)</asp:Content>
<asp:Content runat="server" ContentPlaceHolderID="PageContent">
  <p>
    The requested URL (<span class="url"><%= RequestUrl.PathAndQuery %></span>)
    was not found on this server.</p>
  <p>
    Please review the URL and make sure that it is spelled correctly.</p>
</asp:Content>
<asp:Content runat="server" ContentPlaceHolderID="PageImage">
  <img alt="No entry" src="/Images/icon-do-not-enter-347x346.jpg"
    width="347" height="346" />
</asp:Content>
```

#### 404.aspx.cs

```
using System;
using System.Diagnostics;
using System.Web.UI;

namespace TechnologyToolbox.Caelum.Website.Errors
{
    /// <summary>
    /// Displays a custom message when a "file not found" error occurs.
    /// </summary>
    public partial class Error404Page : Page
    {
        protected Uri RequestUrl { get; private set; }

        /// <summary>
        /// Returns the originally requested URL (by parsing the specified
        /// request URL).
        /// </summary>
        /// <remarks>
        /// When the <customErrors> configuration redirects to this page,
        /// Request.Url contains either the original URL
        /// (e.g. ".../foobar.aspx") or the URL of the 404 error page with the
        /// original URL specified in a query string parameter
        /// (e.g. ".../404.aspx?aspxerrorpath=/blog/jjameson/...") depending on
        /// whether redirectMode="ResponseRewrite" or
        /// redirectMode="ResponseRedirect".
        /// 
        /// However, when a native IIS7 request redirects to this page (e.g.
        /// when requesting "/foobar") then Request.Url is something like
        // ".../Errors/404.aspx?404;https://www.technologytoolbox.com:80/foobar".
        ///
        /// Consequently, a little parsing may be necessary in order to show the
        /// relevant info to the user.
        /// </remarks>
        /// <param name="requestUrl">The URL of the request (i.e. Request.Url).</param>
        /// <returns>The originally requested URL.</returns>
        internal static Uri GetOriginalRequestUrl(
            Uri requestUrl)
        {
            Debug.Assert(requestUrl.IsAbsoluteUri == true);

            if (string.Compare(
                requestUrl.AbsolutePath,
                "/Errors/404.aspx",
                StringComparison.OrdinalIgnoreCase) == 0)
            {
                if (requestUrl.Query.StartsWith(
                    "?404;",
                    StringComparison.OrdinalIgnoreCase) == true)
                {
                    string originalUrl = requestUrl.Query.Substring(
                        "?404;".Length);

                    Uri newUrl;

                    bool success = Uri.TryCreate(
                        originalUrl,
                        UriKind.Absolute,
                        out newUrl);

                    if (success == true)
                    {
                        requestUrl = newUrl;
                    }
                }

                // HACK:
                // This was initially implemented as an "else if", but then I
                // discovered that in some environments (DEV, for example), a
                // "double redirect" occurs to 404.aspx. The first with
                // Request.URL =
                // http://www-dev.technologytoolbox.com/Errors/404.aspx?aspxerrorpath=/blog/jjameson/category/19.aspx
                // and the second with Request.URL =
                // http://www-dev.technologytoolbox.com/Errors/404.aspx?404;http://www-dev.technologytoolbox.com:80/Errors/404.aspx?aspxerrorpath=/blog/jjameson/category/19.aspx
                //
                // By changing this to "if" -- instead of "else if" -- the
                // desired result is achieved regardless of whether one or two
                // redirects occur.
                if (requestUrl.Query.StartsWith(
                    "?aspxerrorpath=",
                    StringComparison.OrdinalIgnoreCase) == true)
                {
                    string originalUrl = requestUrl.Query.Substring(
                        "?aspxerrorpath=".Length);

                    Uri newUrl;

                    // In order to access the PathAndQuery property, we must
                    // have an absolute URL, but the "aspxerrorpath" query
                    // string parameter specifies a server-relative URL.
                    bool success = Uri.TryCreate(
                        originalUrl,
                        UriKind.RelativeOrAbsolute,
                        out newUrl);

                    if (success == true)
                    {
                        if (newUrl.IsAbsoluteUri == false)
                        {
                            newUrl = new Uri(requestUrl, newUrl);
                        }

                        Debug.Assert(newUrl.IsAbsoluteUri == true);
                        requestUrl = newUrl;
                    }
                }
            }

            return requestUrl;
        }

        protected void Page_PreRender(
            object sender,
            EventArgs e)
        {
            RequestUrl = GetOriginalRequestUrl(Request.Url);

            // When a native IIS7 request redirects to this page (e.g.
            // when requesting "/foobar") then there is no underlying
            // HttpException for the master page to retrieve the status code
            // from (and consequently it sets the status code to 500). Therefore
            // explicitly overwrite the status code in the PreRender phase of
            // the page lifecycle.
            Response.StatusCode = 404;
        }
    }
}
```

> **Note**
>
> I typically remove comments from code when pasting into blog posts. However, I've kept all of the comments in this case so that you can understand the details of the implementation (without having to describe them separately).

### Other Errors

The vast majority of errors in your site will be HTTP 500 -- as a result
of unhandled exceptions in code (either due to poor coding techniques or unexpected
scenarios) followed by HTTP 404 errors. That is, of course, unless some hackers
decide to launch a denial-of-service attack on your site -- in which case you
might see a very large volume of 404 errors, 400 (Bad Request), 413 (Request
Entity Too Large), or perhaps even some HTTP 503 (Service Unavailable) errors.

However, barring a DoS attack, you've probably covered 99% of the errors
that might occur on your site using the pieces I've covered thus far.

Keep in mind that authentication and authorization errors in an ASP.NET application
will redirect to your login page by default (via an HTTP 302 response) rather
than generating an HTTP 401 (Unauthorized) error.

However, you might want to consider adding a custom 403 (Forbidden) error
page to handle the scenario where someone attempts to "browse a directory listing."
For example, suppose you create a folder in your application -- such as "/Dashboards"
-- that, for whatever reason, doesn't contain a Default.aspx page. Assuming
you haven't enabled Directory Browsing in IIS, then if someone hacks the URL
to browse to /Dashboards then IIS would show its default 403 error page (similar
to Figure 2). With very little work, you could create your own 403 error page
and "wire it up" via the Web.config file.

