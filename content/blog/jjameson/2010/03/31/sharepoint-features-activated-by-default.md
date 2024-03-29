---
title: SharePoint Features Activated by Default
date: 2010-03-31T04:55:00-06:00
description:
  Here's something interesting I discovered only recently about Microsoft Office
  SharePoint Server (MOSS) 2007 -- even though I've been working with the
  product for years... Any features that are scoped to the WebApplication level
  are automatically activated...
aliases:
  [
    "/blog/jjameson/archive/2010/03/30/sharepoint-features-activated-by-default.aspx",
    "/blog/jjameson/archive/2010/03/31/sharepoint-features-activated-by-default.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/03/31/sharepoint-features-activated-by-default.aspx"
---

Here's something interesting I discovered only recently about Microsoft Office
SharePoint Server (MOSS) 2007 -- even though I've been working with the product
for years...

Any features that are scoped to the **WebApplication** level are automatically
activated by default on any new Web applications.

Actually, the same holds for **Farm**-level features as well (which actually
makes sense in my mind -- although, honestly, I haven't created any
**Farm**-level features so far). However, I was quite surprised to find that
features that I developed for one Web application were subsequently activated
automatically on a new Web application that I created.

A little research yesterday revealed that this behavior is documented on MSDN:

{{< reference title="Feature Element (Feature)"
linkHref="http://msdn.microsoft.com/en-us/library/ms436075.aspx" >}}

Here's the description for the **ActivateOnDefault** attribute of the
**Feature** element:

{{< div-block "fst-italic" >}}

> Optional **Boolean**. **TRUE** if the Feature is activated by default during
> installation or when a Web application is created; **FALSE** if the Feature is
> not activated. This attribute equals **TRUE** by default. The
> **ActivateOnDefault** attribute does not apply to site collection (**Site**)
> or Web site (**Web**) scoped Features.
>
> In general, **Farm**-scoped Features become activated during installation, and
> when a new Web application is created, all installed **Web
> application**-scoped Features in it become activated.

{{< /div-block >}}

From now on, whenever I create a feature scoped to **WebApplication**, I'll be
sure to specify the **ActivateOnDefault** attribute and set it to **FALSE** to
avoid having my features "mysteriously" activated on Web applications that I
don't intend them to be.
