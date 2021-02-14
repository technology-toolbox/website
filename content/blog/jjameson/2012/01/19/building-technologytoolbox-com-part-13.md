---
title: "Serving minified jQuery/CSS in PROD and uncompressed versions in DEV (a.k.a. Building TechnologyToolbox.com, part 13)"
date: 2012-01-19T05:29:51+08:00
excerpt: "In this post, I show how to serve minified versions of JavaScript and CSS files in Production environments and uncompressed versions in Development environments."
draft: true
categories: ["Development", "My System"]
tags: ["Subtext", "Web Development"]
---

In
[my previous post](/blog/jjameson/2012/01/16/building-technologytoolbox-com-part-12), I showed how jQuery and a CSS sprite are used to render
the expandable list under the **Archives **section on the various
blog pages of the Technology Toolbox site. However, I intentionally omitted
details about referencing the jQuery scripts and CSS. In this post, I'll show
how to serve minified versions of JavaScript and CSS files in Production environments
(PROD) and uncompressed versions in Development environments (DEV).

When you create a new **ASP.NET Web Application** in Visual
Studio 2010, the **Scripts** folder contains the following files
by default:

- jquery-1.4.1-vsdoc.js
- jquery-1.4.1.js
- jquery-1.4.1.min.js

While you can certainly choose to serve the jQuery script file directly from
your own site, I recommend referencing a copy on a
[CDN](http://en.wikipedia.org/wiki/Content_delivery_network) instead
-- at least for PROD. This not only helps reduce the load on your own servers,
it also greatly increases the probability that users will have already downloaded
the jQuery file (as a result of browsing to another site that references the
same resource).

> **Note**
> 
>       Originally, I thought I needed to reference a local jQuery file during 
>       development in order to get Intellisense while writing JavaScript (i.e. 
>       by referencing a jQuery script residing side-by-side with a corresponding 
>       "vsdoc" file). However, that turned out not to be true. More on that 
>       in a moment.

Also note that version 1.4.1 of jQuery is rather "long in the tooth" these
days -- considering it was released in early 2010. The current release (at the
time of this post, obviously) is 1.7.1.

If you have installed [NuGet](http://nuget.org/), then you can
very quickly download and add the 1.7.1 versions of the jQuery files to your
project simply by running the following command in the
[Package Manager Console](http://docs.nuget.org/docs/start-here/using-the-package-manager-console):

PM&gt; <kbd>Install-Package jQuery</kbd>

Refer to the following resource for more information on the jQuery
NuGet package:

<cite>NuGet Gallery - jQuery</cite>
[http://nuget.org/packages/jQuery](http://nuget.org/packages/jQuery)

Now let's add a little bit of script to a sample page:

```
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="Default.aspx.cs"
  Inherits="Demo._Default" %>

<!DOCTYPE html PUBLIC
"-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
  <title>jQuery Demo</title>
  <script
    src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"
    type="text/javascript"></script>
</head>
<body>
    <form id="form1" runat="server">
    <h1>
      jQuery Demo
    </h1>
    <script type="text/javascript">
      $(document).ready(function () {
        $("h1").fadeOut();
        $("h1").fadeIn();
      });
    </script>
    </form>
</body>
</html>
```

Note that in the example above, I have hardcoded the path to the minified
version of jQuery 1.7.1 hosted by Google.

Let's add a breakpoint on the line calling the `fadeOut()` method
and then press F5. If you subsequently step into the jQuery code, you'll see
that it's not very easy to read (because the script has been optimized for file
size). To make it easier to debug, we should use the uncompressed version instead.
Ah, but how should we go about doing this?

You probably already know that the default Web.config file contains the following:

```
<configuration>
    ...
    <system.web>
        <compilation debug="true">
    ...
    </system.web>
    ...
</configuration>
```

This is necessary in order for "F5 debugging" to work as expected in Visual
Studio.

However the Web.config file for Release builds does not contain the `debug` attribute, due to the following
transform specified in Web.Release.config:

```
<configuration ...>
  <system.web>
    <compilation xdt:Transform="RemoveAttributes(debug)" />
  </system.web>
</configuration>
```

We can leverage this fact to conditionally include different versions of
the jQuery file for Debug and Release builds, as follows:

```
<% if (HttpContext.Current.IsDebuggingEnabled == true) { %>
  <script
    src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.js"
    type="text/javascript"></script>
<% } else { %>
  <script
    src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"
    type="text/javascript"></script>
<% } %>
```

Assuming you use Debug builds in DEV (and
[LOCAL](/blog/jjameson/2009/09/25/development-and-build-environments)) and Release builds in PROD, you now have a much nicer debugging experience
while still serving up minified jQuery script on your live site. Also note that
Intellisense works as expected during development, which I *believe*
is a result of having the jQuery files in the **Scripts** folder
(even though the ASPX or master page isn't actually referencing these versions).

This technique obviously works with your own script files -- and CSS files
as well. For example, you might end up with something like this:

```
<head runat="server">
  ...
  <title>Technology Toolbox</title>
  ...
<% if (HttpContext.Current.IsDebuggingEnabled == true) { %>
  <link href="~/Themes/Theme-1.0/Main.css" rel="stylesheet" type="text/css" />
  <script
    src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.js"
    type="text/javascript"></script>
  <script
    src="<%= ResolveUrl("~/Scripts/Caelum-1.0.1.js") %>"
    type="text/javascript"></script>
<% } else { %>
  <link
    href="~/Themes/Theme-1.0/Theme-1.0.5.min.css" rel="stylesheet"
    type="text/css" />
  <script
    src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"
    type="text/javascript"></script>
  <script
    src="<%= ResolveUrl("~/Scripts/Caelum-1.0.1.min.js") %>"
    type="text/javascript"></script>
<% } %>
  ...
</head>
```

Note that on the Technology Toolbox site, the Main.css file imports other
CSS files (such as Basic.css, which includes various "reset" rules as well as
basic formatting of HTML elements). When the CSS files are minified, they are
combined into a single CSS file (e.g. Theme-1.0.5.min.css). This eliminates
extraneous HTTP requests, while also making it easier to mitigate caching issues
when any of the CSS files are updated. (Refer to
[one of my previous posts](/blog/jjameson/2010/11/16/avoid-issues-with-caching-by-using-quot-theme-versions-quot) if you want to read more about this technique.)

> **Note**
> 
>       If you are wondering why I use the **[ResolveUrl](http://msdn.microsoft.com/en-us/library/system.web.ui.control.resolveurl.aspx)** method with the `<script>` 
>       elements but not the `<link>` 
>       elements, it's simply because it doesn't work otherwise (despite the 
>       presence of the `runat="server"` 
>       attribute in the `<head>` 
>       element).

Using the technique I have presented thus far works just fine in most scenarios,
but there are a couple of potential issues (depending on your specific circumstances):

- If your site needs to support both HTTP and HTTPS, then the references
  to the jQuery script on the CDN should use the same protocol (otherwise
  users may receive warnings in their browsers).
- If you need to support skins (á la Subtext) then you may not
  be able to use inline script (server-side script, obviously -- not client-side
  script). For example, if you try to conditionally include minified script
  files in a custom Subtext blog skin (i.e. by adding it to PageTemplate.ascx)
  then you'll be greeted with a rather nasty error message:

> The Controls collection cannot be modified because the control contains
> code blocks (i.e. &lt;% ... %&gt;).

To avoid these issues, I created a few server controls to render the
`<script>`
and `<link>`
elements.

### CssReference Control

This control replaces `<link>`
elements used to reference a CSS file. Instead of specifying the path using
the `href` attribute, the default
(minified) CSS file is specified using the `CssFile` attribute. A debug (uncompressed)
CSS file can optionally be specified using the `DebugCssFile` attribute.

```
using System;
using System.Web;
using System.Web.UI.HtmlControls;

namespace TechnologyToolbox.Caelum.Website.Controls
{
    public class CssReference : HtmlLink
    {
        public CssReference()
        {
            this.Attributes.Add("rel", "stylesheet");
            this.Attributes.Add("type", "text/css");
        }

        public string CssFile { get; set; }
        public string DebugCssFile { get; set; }

        protected override void OnPreRender(
            EventArgs e)
        {
            base.OnPreRender(e);

            if (this.Context.IsDebuggingEnabled == true)
            {
                this.Href = this.DebugCssFile;
            }
            else
            {
                this.Href = this.CssFile;
            }
        }
    }
}
```

### JavaScriptReference Control

This control replaces `<script>`
elements used to reference a JavaScript file. Instead of specifying the script
path using the `src` attribute,
the default (minified) script file is specified using the `SourceFile` attribute. A debug (uncompressed)
script file can optionally be specified using the `DebugSourceFile` attribute.

```
using System;
using System.Web;
using System.Web.UI;
using System.Web.UI.HtmlControls;

namespace TechnologyToolbox.Caelum.Website.Controls
{
    public class JavaScriptReference : HtmlControl
    {
        public JavaScriptReference() : base("script")
        {
            this.Attributes.Add("type", "text/javascript");
        }
        
        public string SourceFile { get; set; }
        public string DebugSourceFile { get; set; }

        protected override void OnPreRender(
            EventArgs e)
        {
            base.OnPreRender(e);

            if (this.Context.IsDebuggingEnabled == true)
            {
                string path = ResolveUrl(this.DebugSourceFile);

                this.Attributes.Add("src", path);
            }
            else
            {
                string path = ResolveUrl(this.SourceFile);

                this.Attributes.Add("src", path);
            }
        }

        protected override void Render(
            HtmlTextWriter writer)
        {
            if (writer == null)
            {
                throw new ArgumentNullException("writer");
            }

            writer.WriteBeginTag(this.TagName);
            this.RenderAttributes(writer);
            writer.Write(">");
            writer.WriteEndTag(this.TagName);
        }
    }
}
```

Note that the .NET Framework doesn't currently include an "**HtmlScript**"
control (similar to the **[HtmlLink](http://msdn.microsoft.com/en-us/library/system.web.ui.htmlcontrols.htmllink.aspx)**) control. Consequently it takes a tiny bit more code to
render as expected.

### JQueryReference Control

To encapsulate the details of referencing jQuery hosted on a CDN -- as well
as to automatically switch between "http://" and "https://" -- I created the
**JQueryReference** control. Using this control, the `Version` attribute can be specified
instead of the `SourceFile` and
`DebugSourceFile` attributes
(which is still possible since **JQueryReference **inherits from
**JavaScriptReference**). By default, I initialize **Version**
to the current version of jQuery (i.e. 1.7.1).

```
using System;
using System.Globalization;

namespace TechnologyToolbox.Caelum.Website.Controls
{
    [System.Diagnostics.CodeAnalysis.SuppressMessage(
        "Microsoft.Naming",
        "CA1702:CompoundWordsShouldBeCasedCorrectly",
        MessageId = "JQuery")]
    public class JQueryReference : JavaScriptReference
    {
        public JQueryReference()
        {
            this.Version = "1.7.1"; // default to latest version
        }

        public string Version { get; set; }

        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Naming",
            "CA2204:Literals should be spelled correctly",
            MessageId = "jQuery")]
        protected override void OnPreRender(
            EventArgs e)
        {
            if (string.IsNullOrEmpty(this.Version) == true)
            {
                throw new InvalidOperationException(
                    "The jQuery version must be specified.");
            }

            if (string.IsNullOrEmpty(this.SourceFile) == true)
            {
                this.SourceFile = string.Format(
                    CultureInfo.InvariantCulture,
                    "{0}://ajax.googleapis.com/ajax/libs/jquery/{1}/jquery.min.js",
                    this.Page.Request.IsSecureConnection ? "https" : "http",
                    this.Version);
            }

            if (string.IsNullOrEmpty(this.DebugSourceFile) == true)
            {
                this.DebugSourceFile = string.Format(
                    CultureInfo.InvariantCulture,
                    "{0}://ajax.googleapis.com/ajax/libs/jquery/{1}/jquery.js",
                    this.Page.Request.IsSecureConnection ? "https" : "http",
                    this.Version);
            }
                
            base.OnPreRender(e);
        }
    }
}
```

### Updated &lt;head&gt; for ASPX or Master Page

Leveraging these new controls, the corresponding `<head>`
section now looks like this:

```
<head runat="server">
  ...
  <title>Technology Toolbox</title>
  ...
  <caelum:CssReference runat="server"
    CssFile="~/Themes/Theme-1.0/Theme-1.0.5.min.css"
    DebugCssFile="~/Themes/Theme-1.0/Main.css" />
  <caelum:JQueryReference runat="server" />
  <caelum:JavaScriptReference runat="server"
    SourceFile="~/Scripts/Caelum-1.0.1.min.js"
    DebugSourceFile="~/Scripts/Caelum-1.0.1.js" />
  ...
</head>
```

