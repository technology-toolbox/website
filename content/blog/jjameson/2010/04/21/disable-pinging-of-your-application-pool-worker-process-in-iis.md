---
title: Disable Pinging of Your Application Pool Worker Process in IIS
date: 2010-04-21T04:18:00-06:00
description:
  "Yesterday I was doing another \"Knowledge Transfer\" session and before I
  started walking through some code in a debugging session, I took a brief
  detour to show the team how I recommend disabling the \"ping\" functionality
  in IIS for your application pool..."
aliases:
  [
    "/blog/jjameson/archive/2010/04/20/disable-pinging-of-your-application-pool-worker-process-in-iis.aspx",
    "/blog/jjameson/archive/2010/04/21/disable-pinging-of-your-application-pool-worker-process-in-iis.aspx",
  ]
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "Debugging", "Web Development"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/04/21/disable-pinging-of-your-application-pool-worker-process-in-iis.aspx"
---

Yesterday I was doing another "Knowledge Transfer" session and before I started
walking through some code in a debugging session, I took a brief detour to show
the team how I recommend disabling the "ping" functionality in IIS for your
application pool. Note that this recommendation only applies to development
environments -- please don't disable this on your Test or Production
environments.

In case you aren't aware of it, IIS monitors the health of the worker process
for your application pool to ensure that it is responding to requests. If the
worker process doesn't respond in a timely fashion, IIS summarily terminates the
process. Note that this is the default behavior in IIS 7 (Windows Server 2008).

However, if you are stepping through a Web request in the debugger (presumably
in an isolated environment where you aren't affecting other team members), then
it is mighty inconvenient to have IIS terminate the process you are debugging.

Consequently, one of the first things I try to remember to do after creating a
new application pool -- e.g. creating a new Web application in Microsoft Office
SharePoint Server (MOSS) 2007 -- is to change the **Ping Enabled** setting to
**False**.

I just did a quick Bing search and found the following resources (in case you
want more info or detailed steps):

{{< reference
title="Enable Worker Process Pinging for an Application Pool (IIS 7)"
linkHref="http://technet.microsoft.com/en-us/library/cc725836(WS.10).aspx" >}}

{{< reference title="Error: Web site worker process has been terminated by IIS"
linkHref="http://msdn.microsoft.com/en-us/library/bb763108.aspx" >}}

Happy debugging!
