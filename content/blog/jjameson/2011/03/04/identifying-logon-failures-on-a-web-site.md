---
title: Identifying Logon Failures on a Web Site
date: 2011-03-04T05:01:00-07:00
excerpt:
  "Several years ago, while working on the \"Frontier\" project at Agilent
  Technologies , I encountered a scenario where I needed to quickly identify
  logon failures on the site. The Agilent site was (and I believe still is)
  based on Microsoft Office SharePoint..."
aliases:
  [
    "/blog/jjameson/archive/2011/03/03/identifying-logon-failures-on-a-web-site.aspx",
    "/blog/jjameson/archive/2011/03/04/identifying-logon-failures-on-a-web-site.aspx",
  ]
draft: true
categories: ["SharePoint", "Infrastructure"]
tags: ["MOSS 2007", "Infrastructure", "SharePoint 2010"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2011/03/04/identifying-logon-failures-on-a-web-site.aspx"
---

Several years ago, while working on the "Frontier" project at
[Agilent Technologies](http://chem.agilent.com), I encountered a scenario where
I needed to quickly identify logon failures on the site. The Agilent site was
(and I believe still is) based on Microsoft Office SharePoint Server (MOSS)
2007, and used Forms-Based Authentication with a slightly custom version of the
[ActiveDirectoryMembershipProvider](http://msdn.microsoft.com/en-us/library/system.web.security.activedirectorymembershipprovider%28v=VS.80%29.aspx)
(in order to lookup users by email address, instead of forcing customers and
partners to remember their usernames in the Agilent extranet domain).

Whenever a user failed to authenticate, I noticed that ASP.NET wrote a message
to the Application event log on the Web server (Event ID: 1315) in which the
Description contained the phrase:

{{< div-block "fst-italic" >}}

> Membership credential verification failed.

{{< /div-block >}}

In order to help the Operations team diagnose these logon failures, I created a
couple of scripts that simply query the event log based on the Event ID and
Description.

The first script lists all logon failures for a particular server (i.e. one of
the Web servers in the farm), while the second script looks for logon failures
for a particular user (across all Web servers in the farm, which -- at least
back then -- happened to be two servers: WCOSLSC0 and WCOSLSC9).

These scripts are based on a sample I found in the
[Script Center on TechNet](http://technet.microsoft.com/en-us/scriptcenter/default.aspx)
for querying the event log. You should be able to easily modify them to search
for other events (and subsequently parse data from).

{{< div-block "note" >}}

> **Note**
> 
> While I originally created these scripts for a solution based on MOSS 2007,
> you should be able to use them for any site based on ASP.NET. For example,
> this morning I verified the scripts still work as expected with
> [my Fabrikam Demo site based on SharePoint Server 2010](/blog/jjameson/2011/02/25/claims-login-web-part-for-sharepoint-server-2010)
> and configured with claims-based authentication.

{{< /div-block >}}

{{< div-block "note important" >}}

> **Important**
> 
> You should obviously apply discretion when deciding when (and how often) to
> run these scripts against a Production envionment. With a little bit of
> effort, however, you could modify the scripts to work against saved event logs
> copied to another environment for analysis.

{{< /div-block >}}

### ListLogonFailureEventsForServer.vbs

This script searches the specified server's Application event log for ASP.NET
logon failure events and subsequently parses the username from any events that
are found:

```VBScript
Option Explicit

If (WScript.Arguments.Count <> 1) Then
    WScript.Echo("Usage: ListLogonFailureEventsForServer.vbs {server}")    
    WScript.Quit(1)
End If

Dim strServer
strServer = WScript.Arguments(0)

WScript.Echo("Logon failure events for server: " & strServer)

Call ListLogonFailureEventsForServer(strServer)

Sub ListLogonFailureEventsForServer(ByVal strServer)

    Dim objWMIService
    Set objWMIService = GetObject("winmgmts:" _
        & "{impersonationLevel=impersonate}!\\" & strServer & "\root\cimv2")

    Dim colEvents
    Set colEvents = objWMIService.ExecQuery _
        ("Select * from Win32_NTLogEvent" _
        & " Where" _
            & " Logfile = 'Application'" _
            & " and EventCode = '1315'" _
            & " and Message LIKE '%Membership credential verification failed%'")

    WScript.Echo("Number of logon failure events (" & strServer & "): " _
        & colEvents.Count)

    Dim objEvent
    For Each objEvent In colEvents
        Dim index1
        index1 = InStr(objEvent.Message, "Name to authenticate: ")

        Dim strUser

        If (index1 < 0) Then
            strUser = "(Unknown)"
        Else
            index1 = index1 + Len("Name to authenticate: ")

            Dim index2
            index2 = InStr(index1, objEvent.Message, vbCr)

            strUser = Mid(objEvent.Message, index1, index2 - index1)
        End If

        WScript.Echo(objEvent.TimeGenerated & "	" & strUser)
    Next

    WScript.Echo()
End Sub
```

### ListLogonFailureEventsForUser.vbs

This script searches the Application event logs for each server in the farm
looking for ASP.NET logon failure events for the specified username:

```VBScript
Option Explicit

If (WScript.Arguments.Count <> 1) Then
    WScript.Echo("Usage: ListLogonFailureEventsForUser.vbs {username}")    
    WScript.Quit(1)
End If

Dim strUser
strUser = WScript.Arguments(0)

WScript.Echo("Logon failure events for user: " & strUser)

Call ListLogonFailureEventsForUser(strUser, "WCOSLSC0")
Call ListLogonFailureEventsForUser(strUser, "WCOSLSC9")

Sub ListLogonFailureEventsForUser(ByVal strUser, ByVal strServer)

    Dim objWMIService
    Set objWMIService = GetObject("winmgmts:" _
        & "{impersonationLevel=impersonate}!\\" & strServer & "\root\cimv2")

    Dim colEvents
    Set colEvents = objWMIService.ExecQuery _
        ("Select * from Win32_NTLogEvent" _
        & " Where" _
            & " Logfile = 'Application'" _
            & " and EventCode = '1315'" _
            & " and Message LIKE '%Membership credential verification failed%" _
                & "Name to authenticate: " & strUser & "%'")

    WScript.Echo("Number of logon failure events (" & strServer & "): " _
        & colEvents.Count)

    Dim objEvent
    For Each objEvent In colEvents
        WScript.Echo(objEvent.TimeGenerated)
    Next

    WScript.Echo()
End Sub
```
