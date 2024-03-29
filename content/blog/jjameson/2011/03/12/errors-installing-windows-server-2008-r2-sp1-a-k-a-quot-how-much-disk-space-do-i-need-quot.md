---
title: "Errors Installing Windows Server 2008 R2 SP1 (a.k.a. \"How much disk space do I need?!\")"
date: 2011-03-12T04:32:00-07:00
description:
  "A couple of years ago, I wrote a post about issues I encountered deploying
  Windows Server 2008 Service Pack 2 due to insufficient disk space on various
  VMs in the the \"Jameson Datacenter\" (a.k.a. my home lab). This morning I
  encountered similar issues..."
aliases:
  [
    "/blog/jjameson/archive/2011/03/11/errors-installing-windows-server-2008-r2-sp1-a-k-a-quot-how-much-disk-space-do-i-need-quot.aspx",
    "/blog/jjameson/archive/2011/03/12/errors-installing-windows-server-2008-r2-sp1-a-k-a-quot-how-much-disk-space-do-i-need-quot.aspx",
  ]
categories: ["Infrastructure"]
tags: ["Windows Server", "Infrastructure"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2011/03/12/errors-installing-windows-server-2008-r2-sp1-a-k-a-quot-how-much-disk-space-do-i-need-quot.aspx"
---

A couple of years ago, I wrote a post about
[issues I encountered deploying Windows Server 2008 Service Pack 2](/blog/jjameson/2009/06/01/errors-installing-windows-server-2008-sp2)
due to insufficient disk space on various VMs in the the
["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) (a.k.a.
my home lab).

This morning I encountered similar issues when attempting to install Windows
Server 2008 R2 SP1 (which I recently approved on my WSUS server). Apparently the
disk space requirements have gone up -- again. Significantly.

On the development VM (FOOBAR5) that failed to install the Service Pack, I had
5.5 GB of free space before downloading the 903 MB file from my WSUS server.

Evidently I need 7.4 GB of free space in order to install this latest Service
Pack:

{{< reference
title="Documentation for Windows 7 and Windows Server 2008 R2 Service Pack 1 (KB976932)"
linkHref="http://www.microsoft.com/downloads/en/details.aspx?FamilyID=61924cea-83fe-46e9-96d8-027ae59ddc11&displaylang=en" >}}

Maybe it's time to head over to [newegg](http://www.newegg.com) and see if those
terabyte drives are still on sale ;-)
