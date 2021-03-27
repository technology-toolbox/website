---
title: Implementing Google Analytics (a.k.a. Building TechnologyToolbox.com, part 22)
date: 2012-02-03T02:40:38-07:00
excerpt:
  In yesterday's post, I described how I integrated Google Site Search into the
  Technology Toolbox website. This post provides a similar walkthrough for
  implementing Google Analytics.
aliases:
  [
    "/blog/jjameson/archive/2012/02/02/building-technologytoolbox-com-part-22.aspx",
    "/blog/jjameson/archive/2012/02/03/building-technologytoolbox-com-part-22.aspx",
  ]
draft: true
categories: ["Development", "My System"]
tags: ["Web Development"]
---

In [yesterday's post](../02/building-technologytoolbox-com-part-21.aspx), I
described how I integrated Google Site Search into the Technology Toolbox
website. This post provides a similar walkthrough for implementing
[Google Analytics](http://www.google.com/analytics/).

### Step 1: Sign up for Google Analytics

Unlike the options for integrating Google search into your website, there is
_only_ a free version of Google Analytics. All you need in order to sign up is a
Google Account -- and chances are fairly high you already have one of those.

Once you have signed up, Google assigns you a unique "tracking code" -- or what
I typically refer to as the "analytics key." For example, the analytics key for
Technology Toolbox is UA-25915894-1.

Don't worry, there's nothing secret about these keys; you can view them on any
site using Google Analytics simply by viewing the HTML source for the page.

### Step 2: Adding the tracking code

After obtaining a tracking code -- er, I mean, analytics key -- I added the
snippet of JavaScript provided by Google that records a little bit of
information for each page request. In the early days of Web analytics, this was
often accomplished using Web beacons (e.g. a clear or 1x1 pixel GIF image), but
now products like Google Analytics and Omniture use script.

I decided early on that I wanted a way to easily disable the Google Analytics
script for a couple of reasons:

- When developing new features or fixing bugs, I do not want the page views to
  be recorded to Google. This data is meaningless and I don't see any sense in
  capturing it.
- Based on my research, I had some concerns about potential performance issues
  with Google Analytics. The original implementation of Google Analytics did not
  run asynchronously and consequently had a negative impact on some websites in
  the beginning. These issues were resolved several years ago, but it still
  seems like a good idea to be able to quickly disable the Google script just in
  case a problem is discovered in the future.

To satisfy this design goal, I created a new
[application setting](http://msdn.microsoft.com/en-us/library/cftf714c.aspx)
named **EnableAnalytics** with a default value of **False**.

Next I added an ASP.NET control named **AnalyticsScript** to encapsulate the
logic to conditionally render the Google script:

```C#
using System;
using System.Web.UI;
using System.Web.UI.WebControls;
using TechnologyToolbox.Caelum.Website.Properties;

namespace TechnologyToolbox.Caelum.Website.Controls
{
    [ToolboxData("<{0}:AnalyticsScript runat=server></{0}:AnalyticsScript>")]
    public class AnalyticsScript : WebControl
    {
        protected override void OnLoad(
            EventArgs e)
        {
            base.OnLoad(e);

            this.Visible = Settings.Default.EnableAnalytics;
        }

        protected override void RenderContents(
            HtmlTextWriter writer)
        {
            if (writer == null)
            {
                throw new ArgumentNullException("writer");
            }

            writer.WriteBeginTag("script");
            writer.WriteAttribute("type", "text/javascript", false);
            writer.Write(HtmlTextWriter.TagRightChar);

            writer.WriteLineNoTabs(
@"    var _gaq = _gaq || [];
    _gaq.push(['_setAccount', 'UA-25899478-1']);
    _gaq.push(['_trackPageview']);

    (function () {
        var ga = document.createElement('script'); ga.type = 'text/javascript'; ga.async = true;
        ga.src = ('https:' == document.location.protocol ? 'https://ssl' : 'http://www') + '.google-analytics.com/ga.js';
        var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(ga, s);
    })();");

            writer.WriteEndTag("script");
        }
    }
}
```

Then I added the new control to the master page:

```ASP.NET
<%@ Master ... %>
<%@ Register Tagprefix="caelum"
  Namespace="TechnologyToolbox.Caelum.Website.Controls"
  Assembly="TechnologyToolbox.Caelum.Website, Version=1.0.0.0,
    Culture=neutral, PublicKeyToken=f55b5d7768fcda39" %>
...
<html ...>

<head runat="server">
  ...
  <caelum:AnalyticsScript runat="server" />
  <asp:ContentPlaceHolder id="AdditionalHeadContent" runat="server" />
</head>
...
```

At this point, I changed the **EnableAnalytics** setting in the Web.config file
and verified that everything worked as expected. It was close, but not quite
right...

### Step 3: From WebControl to Control

When I examined the source of the page at this point, I noticed the `<script>`
element was wrapped in a `<span>` element. Oops.

To resolve this, I changed the class to inherit from **Control** (rather than
**WebControl**) and renamed the **RenderContents** method to **Render**.

{{< div-block "note" >}}

> **Note**
> 
> I've used other techniques in the past to eliminate extraneous markup --
> specifically overriding the **RenderBeginTag** and **RenderEndTag** methods.
> In a followup post I will explain why I used a different approach for this
> scenario.

{{< /div-block >}}

At this point, the script rendered as expected -- provided I remembered to
change the **EnableAnalytics** setting in Web.config to **True** (i.e. for
environments other than my local development environment).

### Step 4: Enable analytics by default

Thinking that it would be preferable to enable analytics by default in DEV,
TEST, and PROD -- but still disable it in local development environments -- I
changed the default value for **EnableAnalytics** to **True** and added another
application setting to specify a "filter" as an additional check for determining
whether or not to emit the analytics script:

```XML
<configuration>
  ...
  <applicationSettings>
    <TechnologyToolbox.Caelum.Website.Properties.Settings>
      ...
      <setting name="EnableAnalytics" serializeAs="String">
        <value>True</value>
      </setting>
      <setting name="AnalyticsEnvironmentFilter" serializeAs="String">
        <value>^(www)?(-dev)?(-test)?\.?technologytoolbox.com</value>
      </setting>
    </TechnologyToolbox.Caelum.Website.Properties.Settings>
  </applicationSettings>
</configuration>
```

I then updated the **AnalyticsScript** control to compare the URL of the current
request with the filter specified in configuration:

```C#
        protected override void OnLoad(
            EventArgs e)
        {
            base.OnLoad(e);

            bool enableAnalytics = Settings.Default.EnableAnalytics;

            if (enableAnalytics == true)
            {
                string pattern = Settings.Default.AnalyticsEnvironmentFilter;

                enableAnalytics = Regex.IsMatch(
                    this.Page.Request.Url.Host,
                    pattern);
            }

            this.Visible = enableAnalytics;
        }
```

At this point, I also removed the hard-coded analytics key by adding another
application setting in Web.config:

```XML
<configuration>
  ...
  <applicationSettings>
    <TechnologyToolbox.Caelum.Website.Properties.Settings>
      ...
      <setting name="AnalyticsKey" serializeAs="String">
        <value>UA-25899478-1</value>
      </setting>
    </TechnologyToolbox.Caelum.Website.Properties.Settings>
  </applicationSettings>
</configuration>
```

After verifying the functionality in DEV (www-dev.technologytoolbox.com) and
TEST (www-test.technologytoolbox.com), I deployed the new build to Production
and started seeing results in the Google Analytics dashboard the next day.

### Step 5: Improve the solution for DEV and TEST

Even though the data captured from the development and test environments was
very small -- and thus would not skew the "real" data over time -- I was still a
little concerned with this "junk" data being included in the analytics reports.

{{< div-block "note" >}}

> **Note**
> 
> 
> While researching how other people handled this issue with development and
> test environments, I noticed a number of folks recommending that you filter
> the reports based on the domain name. For example, for Technology Toolbox, I
> could filter out non-Production data by only looking at the
> www.technologytoolbox.com domain name (e.g. exclude
> www-dev.technologytoolbox.com and www-test.technologytoolbox.com).
> 
> I experimented with that approach a little but quickly dismissed it due to the
> enormous effort this would require to customize each of the default reports
> provided by Google.

{{< /div-block >}}

That is when it occurred to me that rather than using a single tracking code for
all environments, I could just as easily use different codes for each
environment, as illustrated in Figure 1.

{{< figure src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Google-Analytics-Account-Home-600x340.png" alt="Google Analytics (Account Home)" height="340" width="600" title="Figure 1: Google Analytics (Account Home)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Google-Analytics-Account-Home-767x435.png)

After some refactoring and performance optimization, here is the updated
implementation of the **AnalyticsScript** class:

```C#
using System;
using System.Diagnostics;
using System.Web.UI;

namespace TechnologyToolbox.Caelum.Website.Controls
{
    [ToolboxData("<{0}:AnalyticsScript runat=server></{0}:AnalyticsScript>")]
    public class AnalyticsScript : Control
    {
        private static bool __enableAnalytics = false;
        private static string __analyticsKey = null;

        private static bool __initialized = false;
        private static object __lockObject = new object();
        
        protected override void OnLoad(
            EventArgs e)
        {
            base.OnLoad(e);

            if (__initialized == false)
            {
                lock (__lockObject)
                {
                    if (__initialized == false)
                    {
                        __enableAnalytics = AnalyticsHelper.IsAnalyticsEnabled(
                            this.Page.Request);

                        if (__enableAnalytics == true)
                        {
                            __analyticsKey = AnalyticsHelper.GetAnalyticsKey(
                                this.Page.Request);
                        }

                        __initialized = true;
                    }
                }
            }

            this.Visible = __enableAnalytics;
        }

        protected override void Render(
            HtmlTextWriter writer)
        {
            if (writer == null)
            {
                throw new ArgumentNullException("writer");
            }

            Debug.Assert(string.IsNullOrEmpty(__analyticsKey) == false);

            writer.WriteBeginTag("script");
            writer.WriteAttribute("type", "text/javascript", false);
            writer.Write(HtmlTextWriter.TagRightChar);

            writer.WriteLineNoTabs(
@"    var _gaq = _gaq || [];
    _gaq.push(['_setAccount', '" + __analyticsKey + @"']);
    _gaq.push(['_trackPageview']);

    (function () {
        var ga = document.createElement('script'); ga.type = 'text/javascript'; ga.async = true;
        ga.src = ('https:' == document.location.protocol ? 'https://ssl' : 'http://www') + '.google-analytics.com/ga.js';
        var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(ga, s);
    })();");

            writer.WriteEndTag("script");
        }
    }
}
```

The new **AnalyticsHelper** class contains some of the code originally added to
the **AnalyticsScript** control. The original code has also been enhanced to
support different analytics keys for DEV, TEST, and PROD:

```C#
using System;
using System.Globalization;
using System.Text.RegularExpressions;
using System.Web;
using TechnologyToolbox.Caelum.CoreServices.Logging;
using TechnologyToolbox.Caelum.Website.Properties;

namespace TechnologyToolbox.Caelum.Website
{
    public static class AnalyticsHelper
    {
        public static string GetAnalyticsKey(
            HttpRequest request)
        {
            HttpRequestWrapper wrapper = new HttpRequestWrapper(request);
            return GetAnalyticsKey(wrapper);
        }

        public static string GetAnalyticsKey(
            HttpRequestBase request)
        {
            if (request == null)
            {
                throw new ArgumentNullException("request");
            }

            string key = Settings.Default.AnalyticsKey;

            if (string.IsNullOrEmpty(key) == true)
            {
                if (string.Compare(
                    request.Url.Host,
                    "www-dev.technologytoolbox.com",
                    StringComparison.OrdinalIgnoreCase) == 0)
                {
                    key = "UA-25949832-1";
                }
                else if (string.Compare(
                    request.Url.Host,
                    "www-test.technologytoolbox.com",
                    StringComparison.OrdinalIgnoreCase) == 0)
                {
                    key = "UA-25899478-1";
                }
                else if (string.Compare(
                    request.Url.Host,
                    "technologytoolbox.com",
                    StringComparison.OrdinalIgnoreCase) == 0
                    || string.Compare(
                        request.Url.Host,
                        "www.technologytoolbox.com",
                        StringComparison.OrdinalIgnoreCase) == 0)
                {
                    key = "UA-25915894-1";
                }
                else
                {
                    string message = string.Format(
                        CultureInfo.CurrentCulture,
                        "No analytics key specified for hostname ({0}).",
                        request.Url.Host);

                    throw new InvalidOperationException(message);
                }
            }

            return key;
        }

        public static bool IsAnalyticsEnabled(
            HttpRequest request)
        {
            HttpRequestWrapper wrapper = new HttpRequestWrapper(request);
            return IsAnalyticsEnabled(wrapper);
        }

        public static bool IsAnalyticsEnabled(
            HttpRequestBase request)
        {
            if (request == null)
            {
                throw new ArgumentNullException("request");
            }

            Logger.LogDebug("Checking if analytics feature is enabled...");

            bool enableAnalytics = Settings.Default.EnableAnalytics;

            if (enableAnalytics == true)
            {
                string pattern = Settings.Default.AnalyticsEnvironmentFilter;
                
                Logger.LogDebug(
                    CultureInfo.CurrentCulture,
                    "Checking if host ({0}) matches analytics environment"
                        + " filter ({1})...",
                    request.Url.Host,
                    pattern);

                enableAnalytics = Regex.IsMatch(
                    request.Url.Host,
                    pattern);
            }

            Logger.LogDebug(
                CultureInfo.CurrentCulture,
                "Analytics feature is {0}.",
                enableAnalytics ? "enabled" : "disabled");

            return enableAnalytics;
        }
    }
}
```

The implementation still supports the ability to override the analytics key by
specifying a value in the Web.config file (for example, if I wanted to record
metrics for some other environment). In my Web.config files, however, the
**AnalyticsKey** setting is left empty:

```XML
<configuration>
  ...
  <applicationSettings>
    <TechnologyToolbox.Caelum.Website.Properties.Settings>
      ...
      <setting name="EnableAnalytics" serializeAs="String">
        <value>True</value>
      </setting>
      <setting name="AnalyticsEnvironmentFilter" serializeAs="String">
        <value>^(www)?(-dev)?(-test)?\.?technologytoolbox.com</value>
      </setting>
      <setting name="AnalyticsKey" serializeAs="String">
        <value />
      </setting>
    </TechnologyToolbox.Caelum.Website.Properties.Settings>
  </applicationSettings>
</configuration>
```

An alternative would be to store a mapping of domain names and analytics keys in
Web.config (instead of hard-coding the values for DEV, TEST, and PROD).
Personally, I don't believe this is worth the additional effort to implement.
