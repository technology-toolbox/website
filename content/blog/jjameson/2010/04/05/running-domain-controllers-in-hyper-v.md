---
title: Running Domain Controllers in Hyper-V
date: 2010-04-05T15:29:00-06:00
description:
  "In a previous post , I noted how the \" Jameson Datacenter \" (a.k.a. my home
  lab) currently runs two domain controllers (DCs) on a couple of VMs. If you
  choose to virtualize one or more DCs in your environment, make sure you are
  aware of the considerations..."
aliases:
  [
    "/blog/jjameson/archive/2010/04/05/running-domain-controllers-in-hyper-v.aspx",
  ]
categories: ["Infrastructure"]
tags: ["Windows Server", "Infrastructure", "Virtualization"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/04/05/running-domain-controllers-in-hyper-v.aspx"
---

In a
[previous post](/blog/jjameson/2008/11/05/server-core-installation-accessing-windows-in-notification-period),
I noted how the
"[Jameson Datacenter](/blog/jjameson/2009/09/14/the-jameson-datacenter)" (a.k.a.
my home lab) currently runs two domain controllers (DCs) on a couple of VMs.

If you choose to virtualize one or more DCs in your environment, make sure you
are aware of the considerations, risks, and recommendations detailed in the
following TechNet guide:

{{< reference title="Running Domain Controllers in Hyper-V"
linkHref="http://technet.microsoft.com/en-us/library/dd363553(WS.10).aspx" >}}

This appears to be the most recent version of the document that I used when
setting up my environment.
