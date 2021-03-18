---
title: Error Installing KB 970892 When Reporting Services Configured with Domain Account
date: 2009-12-07T21:11:00-07:00
excerpt:
  "For a little over a month, Windows Update was failing on one of the servers
  in the \"Jameson Datacenter\" (a.k.a. my home lab). Specifically, KB 970892
  simply would not install on JUBILEE -- my Systems Center Operations Manager
  (SCOM) 2007 VM, that I use..."
aliases:
  [
    "/blog/jjameson/archive/2009/12/07/error-installing-kb-970892-when-reporting-services-configured-with-domain-account.aspx",
  ]
draft: true
categories: ["Infrastructure"]
tags: ["SQL Server", "Infrastructure"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/12/07/error-installing-kb-970892-when-reporting-services-configured-with-domain-account.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/12/07/error-installing-kb-970892-when-reporting-services-configured-with-domain-account.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

For a little over a month, Windows Update was failing on one of the servers in
the ["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter)
(a.k.a. my home lab). Specifically,
[KB 970892](http://support.microsoft.com/kb/970892) simply would not install on
JUBILEE -- my Systems Center Operations Manager (SCOM) 2007 VM, that I use to
monitor a number of physical and virtual machines.

Here's the first event I found regarding this error:

{{< log-excerpt >}}

<samp> Log Name: Application<br>Source: MsiInstaller<br>Date: 10/27/2009 3:03:49
AM<br>Event ID: 10005<br>Task Category: None<br>Level: Error<br>Keywords:
Classic<br>User: SYSTEM<br>Computer:
jubilee.corp.technologytoolbox.com<br>Description:<br>Product: Microsoft SQL
Server 2005 Reporting Services (64-bit) -- Error 29528. The setup has
encountered an unexpected error while Setting reporting service and share point
exclusion path. The error is: Fatal error during installation.</samp>

{{< /log-excerpt >}}

This was quickly followed by another error in the event log:

{{< log-excerpt >}}

<samp> Log Name: Application<br>Source: MsiInstaller<br>Date: 10/27/2009 3:04:55
AM<br>Event ID: 1023<br>Task Category: None<br>Level: Error<br>Keywords:
Classic<br>User: SYSTEM<br>Computer:
jubilee.corp.technologytoolbox.com<br>Description:<br>Product: Microsoft SQL
Server 2005 Reporting Services (64-bit) - Update 'GDR 4053 for SQL Server
Reporting Services 2005 (64-bit) ENU (KB970892)' could not be installed. Error
code 1603. Additional information is available in the log file C:\Program
Files\Microsoft SQL Server\90\Setup
Bootstrap\LOG\Hotfix\RS9_Hotfix_KB970892_sqlrun_rs.msp.log.</samp>

{{< /log-excerpt >}}

Shortly thereafter, I started seeing the following error once every minute:

{{< log-excerpt >}}

<samp> Log Name: Application<br>Source: Report Server (MSSQLSERVER)<br>Date:
10/27/2009 3:06:52 AM<br>Event ID: 107<br>Task Category: Management<br>Level:
Error<br>Keywords: Classic<br>User: N/A<br>Computer:
jubilee.corp.technologytoolbox.com<br>Description:<br>Report Server
(MSSQLSERVER) cannot connect to the report server database.</samp>

{{< /log-excerpt >}}

Since I have Windows Update configured to automatically download and install
updates every morning, the patch attempted to install each day -- but failed
each and every time.

I have to admit that I've spent a fair amount of time troubleshooting this error
over the past month, but since it wasn't a blocking issue -- just a particularly
irritating annoyance -- I kept putting it off. [Honestly, I rarely look at the
SCOM reports and instead rely mostly on email notifications and the Operations
Manager Console.]

Fortunately, I finally managed to determine the root cause tonight and resolve
the issue.

After downloading and installing the standalone patch installation, I discovered
the following in the installation log:

{{< log-excerpt >}}

```
MSI (s) (F0:54) [21:09:49:565]: Invoking remote custom action. DLL: C:\Windows\Installer\MSIAC31.tmp, Entrypoint: Do_RSSetSharePointExclusionPath
<Func Name='LaunchFunction'>
Function=Do_RSSetSharePointExclusionPath
<Func Name='GetCAContext'>
<EndFunc Name='GetCAContext' Return='T' GetLastError='203'>
Doing Action: Do_RSSetSharePointExclusionPath
PerfTime Start: Do_RSSetSharePointExclusionPath : Mon Dec 07 21:09:49 2009
<Func Name='Do_RSSetSharePointExclusionPath'>
The application pool /s already exists.
Error Code: 0x80077374 (29556)
Windows Error Text: Source File Name: sqlca\sqliisca.cpp
Compiler Timestamp: Mon Nov 17 17:05:40 2008
Function Name: Do_RSSetSharePointExclusionPath
Source Line Number: 914
```

{{< /log-excerpt >}}

As noted in [KB 917826](http://support.microsoft.com/kb/917826), there appears
to be a known issue when Reporting Services is configured to run using a domain
account. For JUBILEE, the **ReportServer** application pool was configured to
run as **TECHTOOLBOX\svc-mom-das** (the SCOM data access service account). After
changing the app pool to run as **NetworkService** instead, I ran the standalone
install of KB 970892 and it completed successfully.

I then changed the app pool identity back to TECHTOOLBOX\svc-mom-das (since that
appears to be how SCOM 2007 wants it configured) and verified that a couple of
reports run successfully. Woohoo!

I'm crossing my fingers that tomorrow morning, Windows Update detects that KB
970892 is installed and no errors occur.
