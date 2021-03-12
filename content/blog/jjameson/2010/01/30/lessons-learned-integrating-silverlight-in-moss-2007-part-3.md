---
title: "Lessons Learned Integrating Silverlight in MOSS 2007, Part 3"
date: 2010-01-30T06:20:00-07:00
excerpt: "Yesterday I continued building upon part 1 in a series of posts regarding the use of Silverlight in an Internet-facing customer portal built on Microsoft Office SharePoint Server (MOSS) 2007. 
 As I mentioned in the previous posts, the Silverlight application..."
aliases: ["/blog/jjameson/archive/2010/01/29/lessons-learned-integrating-silverlight-in-moss-2007-part-3.aspx", "/blog/jjameson/archive/2010/01/30/lessons-learned-integrating-silverlight-in-moss-2007-part-3.aspx"]
draft: true
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "WSS v3", "Silverlight"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/01/30/lessons-learned-integrating-silverlight-in-moss-2007-part-3.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/01/30/lessons-learned-integrating-silverlight-in-moss-2007-part-3.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

[Yesterday](/blog/jjameson/2010/01/29/lessons-learned-integrating-silverlight-in-moss-2007-part-2)
I continued building upon
[part 1](/blog/jjameson/2010/01/28/lessons-learned-integrating-silverlight-in-moss-2007-part-1)
in a series of posts regarding the use of Silverlight in an Internet-facing
customer portal built on Microsoft Office SharePoint Server (MOSS) 2007.

As I mentioned in the previous posts, the Silverlight application is hosted
inside of the SharePoint site by a user control (**ServiceWheel.ascx**) which
itself is hosted within a generic *User Control Web Part* (similar to
[SmartPart for SharePoint](http://www.codeplex.com/smartpart)).

I also mentioned that the user control originally contained the following code:

```
    <object data="data:application/x-silverlight-2," type="application/x-silverlight-2"
        width="100%" height="100%">
        <param name="source" value="_Layouts/Fabrikam/Wheel.xap" />
        <param name="onError" value="onSilverlightError" />
        <param name="background" value="white" />
        <param name="minRuntimeVersion" value="3.0.40624.0" />
        <param name="autoUpgrade" value="true" />
        <a href="http://go.microsoft.com/fwlink/?LinkID=149156&v=3.0.40624.0" style="text-decoration: none">
            <img src="http://go.microsoft.com/fwlink/?LinkId=108181" alt="Get Microsoft Silverlight"
                style="border-style: none" />
        </a>
    </object>
```

In addition to the two problems with this code that I covered in my previous
post, I mentioned there was another issue with the original declaration of the
`<object>` element for the Silverlight control.

Here's a partial screenshot of the home page of the portal, after clicking the
**Site Actions** menu and then clicking **Edit Page**.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/Silverlight-No-menu-open-447x300.png"
alt="No menus open" class="screenshot" height="300" width="447"
title="Figure 1: No menus open" >}}

While the above screenshot doesn't illustrate any problem, the caption gives you
a hint as to what's coming next.

The following screenshot shows what happens when you click the **Page** menu.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/Silverlight-Obscured-menu-447x300.png"
alt="Page menu items obscured by Silverlight control" class="screenshot"
height="300" width="447"
title="Figure 2: Page menu items obscured by Silverlight control" >}}

Good luck trying to click the **Delete Page**, **Add Web Parts**, and **Modify
Web Parts** menu items!

Note that menu items on the **Workflow** and **Tools** menus are similarly
obscured.

Fortunately, once I discovered this problem, it didn't take long to find a
solution. The trick is to set the
[Windowless](http://msdn.microsoft.com/en-us/library/cc838156%28VS.95%29.aspx)
property to **true** in the `<object>` element, as illustrated in the following
screenshot:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/Silverlight-Unobscured-menu-(windowless)-447x300.png"
alt="Page menu items no longer obscured by windowless Silverlight control"
class="screenshot" height="300" width="447"
title="Figure 3: Page menu items no longer obscured by windowless Silverlight
control" >}}

Here's a comment from Karl Erickson on his blog post entitled "
[Limitations of Windowless mode for Silverlight](http://blogs.msdn.com/silverlight_sdk/archive/2008/11/12/limitations-of-windowless-mode-for-silverlight.aspx)":

{{< blockquote "font-italic" >}}

Windowless mode is the only way to create user interfaces that blend HTML and
Silverlight, regardless of which one is on top. Without Windowless mode, the
Silverlight plug-in has its own window, which is always on top, and cannot blend
in HTML UI from underneath.

{{< /blockquote >}}

The latest version of the markup in the user control that hosts our Silverlight
application is shown below:

```
    <object data="data:application/x-silverlight-2," type="application/x-silverlight-2"
        width="380px" height="410px" onFocus="this.style.outline='none';">
        <param name="source" value="<%= serviceWheelPackageUrl %>" />
        <param name="onError" value="onSilverlightError" />
        <param name="background" value="white" />
        <param name="minRuntimeVersion" value="3.0.40624.0" />
        <param name="autoUpgrade" value="true" />
        <param name="windowless" value="true" />
        <a href="http://go.microsoft.com/fwlink/?LinkID=149156&v=3.0.40624.0" style="text-decoration: none">
            <img src="/_layouts/Images/Fabrikam/InstallSilverlight.png" alt="Get Microsoft Silverlight"
                style="border-style: none" />
        </a>
    </object>
```

