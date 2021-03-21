---
title: AutoEventWireup Issue in MOSS 2007
date: 2009-11-08T18:15:00-07:00
excerpt:
  "I recently promised to finish this blog post that has been sitting in
  \"unpublished\" status since June 2008, so here it is... Have you ever
  encountered the following error in Microsoft Office SharePoint Server (MOSS)
  2007? An error occurred during..."
aliases:
  [
    "/blog/jjameson/archive/2009/11/08/autoeventwireup-issue-in-moss-2007.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/11/08/autoeventwireup-issue-in-moss-2007.aspx"
---

I [recently promised](/blog/jjameson/2009/11/02/analyzing-my-msdn-blog) to
finish this blog post that has been sitting in "unpublished" status since June
2008, so here it is...

Have you ever encountered the following error in Microsoft Office SharePoint
Server (MOSS) 2007?

{{< blockquote "fst-italic text-danger" >}}

An error occurred during the processing of . The attribute 'autoeventwireup' is
not allowed in this page.

{{< /blockquote >}}

I just
[searched for this](http://www.bing.com/search?q=SharePoint+%22AutoEventWireup+is+not+allowed%22&form=QBRE&qs=n)
using Bing and it seems like I'm not the only one who has ever experienced this
issue. However, glancing through a few of the top search results, I didn't see
any solutions to the error.

The problem occurs when you have a custom master page which includes code and
that master page subsequently becomes unghosted. I believe this happens with
custom page layouts that are customized as well.

I have to admit that I was completely stumped when I first encountered this
error a few years ago while working on the Agilent Technologies project. I
eventually tracked down the root cause to be unghosted pages, but we were not
using SharePoint Designer to create or customize our master pages, so I couldn't
understand why we would occasionally encounter this error.

My speculation is that when the feature/solution containing the custom master
page is deactivated, retracted, and deleted (as part of the
["DR.DADA" process](/blog/jjameson/2009/03/31/introducing-the-dr-dada-approach-to-sharepoint-development)),
SharePoint has some "smarts" within it that essentially equates to:

- Hey, this master page (or page layout) is currently in use so removing it
  could really break the site.
- Therefore, I'd better make a copy of it and store it in the database (i.e.
  unghost it).

Unfortunately, when we subsequently added, deployed, and activated the
solution/feature, SharePoint would still attempt to use the unghosted master
page and summarily generate the error stating that "the attribute
'autoeventwireup' is not allowed in this page."

Note that this is pure speculation on my part as to what was causing the master
page to become unghosted.

However, what I do know for sure is that once I reghosted the master page, the
AutoEventWireup error would magically disappear.

Here are the steps to reghost a master page or page layout:

1. Browse to **Site Settings** page for your site. Note that if your master page
   is causing the AutoEventWireup error, you can explicitly specify the URL
   (e.g. http://fabrikam/\_layouts/settings.aspx).
2. On the **Site Settings** page, under the **Look and Feel** section, click
   **Reset to site definition**.
3. On the **Reset Page to Site Definition Version**page:
   1. In the **Reset to Site Definition** section, ensure the option to <label
for="ctl00_PlaceHolderMain_ctl00_ResetOnePage"><strong>Reset specific page
      to site definition version</strong> is selected, and then in t</label>he
      **Local URL of the page** box, <label
for="ctl00_PlaceHolderMain_ctl00_ResetOnePage">typtype the URL of the
      master page or page layout that you want reset (e.g.
      http://fabrikam/_catalogs/masterpage/FabrikamMinimal.master).</label>
   2. Click **Reset**.
   3. In the confirmation dialog that appears stating that you will lose all
      customizations, including web part zones, custom controls, and in-line
      text, click **OK**.

Note that in ASP.NET, the default value for the
[`AutoEventWireup`](http://support.microsoft.com/kb/814745) attribute is true.
Therefore you might assume that you could simply remove the attribute from your
custom master page in order to avoid the error when the master page is
unghosted. After all, the error clearly states that the `AutoEventWireup`
attribute is not allowed in this page, right?

In other words, the solution to the problem would seem to be simply be a matter
of changing something like this...

```ASP.NET
<%@ Master Language="C#" AutoEventWireup="true" Codebehind="FabrikamMinimal.master.cs"
    Inherits="Fabrikam.Demo.Publishing.Layouts.MasterPages.FabrikamMinimal" %>
```

...to this:

```ASP.NET
<%@ Master Language="C#" Codebehind="FabrikamMinimal.master.cs"
    Inherits="Fabrikam.Demo.Publishing.Layouts.MasterPages.FabrikamMinimal" %>
```

Unfortunately -- at least in my experience -- this doesn't work. It only leads
to other errors, such as:

{{< blockquote "fst-italic text-danger" >}}

The event handler 'OnPreRender' is not allowed in this page.

{{< /blockquote >}}

The above error occurs when the master page contains something like the
following:

```ASP.NET
            <asp:SiteMapPath ID="BreadcrumbSiteMapPath" Runat="server"
                SiteMapProvider="CurrentNavSiteMapProviderNoEncode"
                RenderCurrentNodeAsLink="true"
                SkipLinkText=""
                OnPreRender="BreadcrumbSiteMapPath_OnPreRender">
```

I attempted to resolve this by converting the
`BreadcrumbSiteMapPath_OnPreRender` event handler to a method and invoking the
method from the `Page_PreRender` event handler instead. However, that only led
to yet another error:

{{< blockquote "fst-italic text-danger" >}}

Code blocks are not allowed in this file.

{{< /blockquote >}}

Sensing a very deep "rat hole" at this point, I decided it wasn't worth pursuing
this issue any further.

Fortunately, as I've stated before, I don't believe master pages and page
layouts deployed through solutions and features should subsequently be
customized through SharePoint Designer. In my opinion, these items should be
tightly managed through your SCM (software configuration management) process --
in other words, versioned in your source control system and subsequently
deployed through a formal change process.

Of course, if your custom master pages and page layouts are very simple (i.e. no
code-behind) then you probably will never encounter this problem.
