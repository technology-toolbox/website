---
title: "SharePoint Web Part to Redirect from HTTP to HTTPS"
date: 2009-11-10T05:09:00-07:00
excerpt:
  "Yesterday, I detailed the steps I recommend for configuring SSL on sites
  built on Microsoft Office SharePoint Server (MOSS) 2007. I also mentioned that
  users won't automatically be redirected from HTTP to HTTPS, and how I've
  previously used a little bit..."
aliases:
  [
    "/blog/jjameson/archive/2009/11/09/sharepoint-web-part-to-redirect-from-http-to-https.aspx",
    "/blog/jjameson/archive/2009/11/10/sharepoint-web-part-to-redirect-from-http-to-https.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/11/10/sharepoint-web-part-to-redirect-from-http-to-https.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/11/10/sharepoint-web-part-to-redirect-from-http-to-https.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Yesterday, I detailed the
[steps I recommend for configuring SSL](/blog/jjameson/2009/11/09/configuring-ssl-on-sharepoint-sites)
on sites built on Microsoft Office SharePoint Server (MOSS) 2007. I also
mentioned that users won't automatically be redirected from HTTP to HTTPS, and
how I've previously used a little bit of code to automatically perform this
redirection.

Here is a base class that contains the core logic for detecting when a redirect
from HTTP to HTTPS is required and automatically redirecting as necessary:

```
using System;
using System.Diagnostics;
using System.Globalization;
using System.Text;
using System.Web;
using System.Web.UI.WebControls.WebParts;

using Microsoft.SharePoint;
using Microsoft.SharePoint.WebControls;

using Fabrikam.Demo.CoreServices.Logging;
using Fabrikam.Demo.Web.Properties;

namespace Fabrikam.Demo.Web.UI.WebControls
{
    /// <summary>
    /// Base class for Web Parts that require secure communication (i.e. HTTPS).
    /// </summary>
    public abstract class SslRequiredWebPart : WebPart
    {
        /// <summary>
        /// Redirect from HTTP to HTTPS (if required).
        /// </summary>
        /// <param name="e">An <see cref="System.EventArgs" /> object that
        /// contains the event data.</param>
        protected override void OnInit(
            EventArgs e)
        {
            base.OnInit(e);

            HttpContext context = HttpContext.Current;

            if (context == null)
            {
                throw new InvalidOperationException(
                    "HttpContext.Current is null.");
            }

            HttpContextWrapper contextWrapper = new HttpContextWrapper(context);

            bool forceRedirect = IsSslRedirectRequired(contextWrapper);

            if (forceRedirect == false)
            {
                return;
            }

            HttpRequest request = context.Request;

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "SslRequiredWebPart - Redirecting from HTTP to HTTPS"
                    + " for URL ({0})...",
                request.RawUrl);

            Debug.Assert(request.RawUrl.StartsWith(
                "/",
                StringComparison.OrdinalIgnoreCase) == true);

            StringBuilder urlBuilder = new StringBuilder();

            urlBuilder.Append(Uri.UriSchemeHttps);
            urlBuilder.Append(Uri.SchemeDelimiter);
            urlBuilder.Append(request.Url.Host);
            urlBuilder.Append(request.RawUrl);

            string redirectUrl = urlBuilder.ToString();

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "SslRequiredWebPart - Redirecting to URL ({0})...",
                redirectUrl);

            context.Response.Redirect(redirectUrl, false);
            context.ApplicationInstance.CompleteRequest();
        }

        /// <summary>
        /// Determines if an SSL redirect is required.
        /// </summary>
        /// <param name="context">Context information representing an individual
        /// HTTP request.</param>
        /// <returns><c>true</c> if a redirect from HTTP to HTTPS is required,
        /// otherwise <c>false</c>.</returns>
        protected virtual bool IsSslRedirectRequired(
            HttpContextBase context)
        {
            if (context == null)
            {
                throw new ArgumentNullException("context");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "SslRequiredWebPart - RedirectToHttpsWhenSslRequired = {0}",
                Settings.Default.RedirectToHttpsWhenSslRequired);

            if (Settings.Default.RedirectToHttpsWhenSslRequired == false)
            {
                return false;
            }

            HttpRequestBase request = context.Request;

            if (request.IsSecureConnection == true)
            {
                Logger.LogDebug(
                   CultureInfo.InvariantCulture,
                   "SslRequiredWebPart - The connection is secure.",
                   request.RawUrl);

                return false;
            }

            if (request.Url.Host.Contains(".") == false)
            {
                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "SslRequiredWebPart - The hostname is not a fully-qualified"
                        + " domain name.");

                return false;
            }
            else if (request.Url.Host.Contains("-local.") == true
                || request.Url.Host.Contains("-dev.") == true)
            {
                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "SslRequiredWebPart - LOCAL or DEV environment detected.");

                return false;
            }

            if (SPContext.Current != null)
            {
                SPControlMode formMode = SPContext.Current.FormContext.FormMode;

                if (formMode == SPControlMode.Edit
                    || formMode == SPControlMode.New)
                {
                    // Never redirect when editing a page
                    Logger.LogDebug(
                        CultureInfo.InvariantCulture,
                        "SslRequiredWebPart - SSL redirect is not required"
                        + " because the page is being edited.");

                    return false;
                }
            }

            return true;
        }
    }
}
```

Using this approach, users can browse the anonymous areas of your site using
HTTP. However, as soon as they browse to a page that contains a Web Part that
requires secure communication, they are automatically redirected from HTTP to
HTTPS.

Note how I use a custom application setting (`RedirectToHttpsWhenSslRequired`)
to allow this feature to be turned off. The default value is `True` (meaning the
feature is turned on). However, by changing the property setting to `False` in
the Web.config file, the feature can be disabled as necessary for a particular
environment or server (for troubleshooting purposes):

```
<configuration>
  <configSections>
    <sectionGroup
      name="applicationSettings"
      type="System.Configuration.ApplicationSettingsGroup, System, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" >
      <section name="Fabrikam.Demo.Web.Properties.Settings"
        type="System.Configuration.ClientSettingsSection, System, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089"
        requirePermission="false" />
    </sectionGroup>
  </configSections>
  <applicationSettings>
    <Fabrikam.Demo.Web.Properties.Settings>
      <setting name="RedirectToHttpsWhenSslRequired" serializeAs="String">
        <value>False</value>
      </setting>
    </Fabrikam.Demo.Web.Properties.Settings>
  </applicationSettings>
</configuration>
```

Also note that there are several scenarios in which a redirect is avoided:

- When the connection is already secure (well, duh...we obviously don't want to
  cause an endless redirect loop)
- When the request URL does not specify a fully qualified domain name, but
  rather an intranet URL (e.g. http://fabrikam). In this scenario, user are
  expected to be authenticated using Windows Authentication, which does not send
  credentials in clear text and therefore does not require SSL.
- In LOCAL developer environments (e.g. http://www-local.fabrikam.com) and the
  Development Integration environment (DEV) -- e.g. http://www-dev.fabrikam.com
  -- since these environments don't typically have SSL certificates installed.
  This is an example of why a standard
  [environment naming convention](/blog/jjameson/2009/06/09/environment-naming-conventions)
  is important.
- When the page where the Web Part resides is being edited (because we don't
  want to force a redirect immediately after someone adds the Web Part to a
  page). This scenario is not expected to occur, since content managers will
  typically use the intranet URL (e.g. http://fabrikam) for creating and editing
  pages. However, it is covered just in case the scenario is ever encountered.

Assuming the `IsSslRedirectRequired` method returns `true`, the Web Part
redirects to an HTTPS connection while preserving all of the query string
parameters specified in the original request. Note that this simple approach
only supports an HTTP GET on the original request. In other words, HTTP POST
requests are "not supported" (however they are not expected to occur either).
This is because any form parameters specified in the body of an HTTP POST
request would be dropped in the redirect.

Similar code can also be used in a page (e.g. /\_layouts/Fabrikam/Login.aspx) if
you prefer that approach instead of creating some kind of "Login" Web Part.

Lastly, note that I allow the `IsSslRedirectRequired` method to be overridden in
Web Parts that inherit from `SslRequiredWebPart` (e.g. `LoginFormWebPart`). I
haven't yet found this to be necessary, but it's there just in case.
