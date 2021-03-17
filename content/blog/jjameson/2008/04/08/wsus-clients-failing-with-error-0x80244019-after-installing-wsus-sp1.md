---
title: "WSUS Clients Failing with Error 0x80244019 After Installing WSUS SP1"
date: 2008-04-08T07:04:00-06:00
excerpt: "Last week I discovered that many of my servers were no longer updating successfully from my intranet Windows Server Update Services (WSUS) server. Looking in WindowsUpdate.log file, I noticed errors similar to the following: 
 2008-03-22 18:53:24:377..."
aliases: ["/blog/jjameson/archive/2008/04/07/wsus-clients-failing-with-error-0x80244019-after-installing-wsus-sp1.aspx", "/blog/jjameson/archive/2008/04/08/wsus-clients-failing-with-error-0x80244019-after-installing-wsus-sp1.aspx"]
draft: true
categories: ["Infrastructure"]
tags: ["WSUS"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2008/04/08/wsus-clients-failing-with-error-0x80244019-after-installing-wsus-sp1.aspx](http://blogs.msdn.com/b/jjameson/archive/2008/04/08/wsus-clients-failing-with-error-0x80244019-after-installing-wsus-sp1.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Last week I discovered that many of my servers were no longer updating
successfully from my intranet Windows Server Update Services (WSUS) server.
Looking in WindowsUpdate.log file, I noticed errors similar to the following:

{{< log-excerpt >}}

```
2008-03-22 18:53:24:377 808 ba0 Misc WARNING: WinHttp: SendRequestToServerForFileInformation failed with 0x80190194
2008-03-22 18:53:24:377 808 ba0 Misc WARNING: WinHttp: ShouldFileBeDownloaded failed with 0x80190194
2008-03-22 18:53:24:377 808 ba0 Misc WARNING: DownloadFileInternal failed for http://colossus:8530/selfupdate/wuident.cab: error 0x80190194
2008-03-22 18:53:24:377 808 ba0 Setup FATAL: IsUpdateRequired failed with error 0x80244019
2008-03-22 18:53:24:377 808 ba0 Setup WARNING: SelfUpdate: Default Service: IsUpdateRequired failed: 0x80244019
2008-03-22 18:53:24:377 808 ba0 Setup WARNING: SelfUpdate: Default Service: IsUpdateRequired failed, error = 0x80244019
2008-03-22 18:53:24:377 808 ba0 Agent * WARNING: Skipping scan, self-update check returned 0x80244019
2008-03-22 18:53:25:002 808 ba0 Agent * WARNING: Exit code = 0x80244019
```

{{< /log-excerpt >}}

Upon troubleshooting the problem, I discovered that the **SelfUpdate** virtual
directory was not configured on my WSUS server (i.e. the server named COLOSSUS).
To resolve the issue, I used IIS Manager to create the **SelfUpdate** virtual
directory (using the local path **C:\Program Files\Update
Services\Selfupdate**). I then used {{< kbd "wuauclt /detectnow" >}} to force an
update on one of my servers to confirm that this resolved the issue.

A little post mortem analysis further revealed the following event on COLOSSUS:

{{< blockquote "font-italic" >}}

Event Type: Information\
Event Source: MsiInstaller\
Event Category: None\
Event ID: 11724\
Date: 3/22/2008\
Time: 7:17:29 AM\
Computer: COLOSSUS\
Description:\
Product: Microsoft Windows Server Update Services 3.0 -- Removal completed
successfully.

{{< /blockquote >}}

Ah, yes...now it was all coming back to me. On the morning of March 22nd, I
decided to install WSUS Service Pack 1 (SP1). Since I did not encounter any
errors during the install, it appears that WSUS SP1 is the likely culprit for
the missing **SelfUpdate** virtual directory.

Perhaps this only happens when you have WSUS installed on a non-standard port or
when you have SharePoint installed on the WSUS server. As you can see in the
excerpt from WindowsUpdate.log above, I am using port 8530 for the WSUS Web
site. This is because COLOSSUS is currently running both WSUS and MOSS 2007.
