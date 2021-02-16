---
title: "Running Domain Controllers in Hyper-V"
date: 2010-04-05T09:29:00-07:00
excerpt: "In a previous post , I noted how the \" Jameson Datacenter \" (a.k.a. my home lab) currently runs two domain controllers (DCs) on a couple of VMs. 
 If you choose to virtualize one or more DCs in your environment, make sure you are aware of the considerations..."
draft: true
categories: ["Infrastructure"]
tags: ["Windows Server", "Infrastructure", "Virtualization"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/05/running-domain-controllers-in-hyper-v.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/05/running-domain-controllers-in-hyper-v.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

In a [previous post](/blog/jjameson/2008/11/05/server-core-installation-accessing-windows-in-notification-period), I noted how the "[Jameson Datacenter](/blog/jjameson/2009/09/14/the-jameson-datacenter)" (a.k.a. my home lab) currently runs two domain controllers (DCs) on a couple of VMs.

If you choose to virtualize one or more DCs in your environment, make sure you are aware of the considerations, risks, and recommendations detailed in the following TechNet guide:

{{< reference title="Running Domain Controllers in Hyper-V" linkHref="http://technet.microsoft.com/en-us/library/dd363553(WS.10).aspx" >}}

This appears to be the most recent version of the document that I used when setting up my environment.

