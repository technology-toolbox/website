---
title: Script to Clear (and Save) Event Logs
date: 2011-03-01T05:37:00-07:00
description:
  "As I was writing my first post earlier this morning , I wondered if I had
  previously shared the script I use to quickly clear the event logs on a server
  (but saving them first -- just in case I need to go back and retrieve
  something from the \"archive..."
aliases:
  [
    "/blog/jjameson/archive/2011/02/28/script-to-clear-and-save-event-logs.aspx",
    "/blog/jjameson/archive/2011/03/01/script-to-clear-and-save-event-logs.aspx",
  ]
categories: ["Infrastructure", "My System"]
tags: ["Windows Server", "Windows 7", "Toolbox"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2011/03/01/script-to-clear-and-save-event-logs.aspx"
---

As I was writing
[my first post earlier this morning](/blog/jjameson/2011/03/01/script-to-restart-sharepoint-2010-services),
I wondered if I had previously shared the script I use to quickly clear the
event logs on a server (but saving them first -- just in case I need to go back
and retrieve something from the "archive").

I did a quick search on my blog and didn't see anything, so I figured that I
should create a quick post to share this with others who might find it useful.
If memory serves, the following script was something I put together based on
various samples from the TechNet Script Center.

Note that I typically use this script only in development environments. I
discovered a few years ago that Operations Manager doesn't like it when I clear
the event logs on "Production" servers in the
"[Jameson Datacenter](/blog/jjameson/2009/09/14/the-jameson-datacenter)" (a.k.a.
my home lab). It's not that anything really bad happens, but rather the
Operations Manager agent detects the event logs have been cleared and
subsequently generates a warning. In other words, it's probably not considered a
best practice to clear your event logs on a server that is actively being
monitored by something like Operations Manager.

Here is the script from my
[Toolbox](/blog/jjameson/2007/03/22/backedup-and-notbackedup) folder
(\NotBackedUp\Public\Toolbox\Scripts\Clear Event Logs.vbs):

```VBA
If WScript.Arguments.Count > 1 Then
    WScript.Echo
    WScript.Echo "Usage: cscript ""Clear Event Logs.vbs"" [computer name]"
    WScript.Echo
    WScript.Quit
End If

Dim strComputer ' As String

If WScript.Arguments.Count > 0 Then
    strComputer= WScript.Arguments(0)
Else
    strComputer= "localhost"
End If

ClearEventLogs strComputer

WScript.Echo "Done"

Private Sub ClearEventLogs( _
    strComputer)

    WScript.Echo "Clearing event logs on " & strComputer & "..."

    Set objWMIService = GetObject( _
        "winmgmts:" & "{impersonationLevel=impersonate,(Backup)}!\\" _
            & strComputer & "\root\cimv2")

    Set colLogFiles = objWMIService.ExecQuery( _
        "Select * from Win32_NTEventLogFile")

    For Each objLogfile in colLogFiles
        ClearEventLog strComputer, objLogfile.LogfileName
    Next
End Sub

Private Sub ClearEventLog( _
    strComputer, _
    strEventLogName)

    WScript.Echo "Clearing '" & strEventLogName & "' event log on " _
        & strComputer & "..."

    Set objWMIService = GetObject( _
        "winmgmts:" & "{impersonationLevel=impersonate,(Backup)}!\\" _
            & strComputer & "\root\cimv2")

    Set colLogFiles = objWMIService.ExecQuery( _
        "Select * from Win32_NTEventLogFile where LogFileName='" _
            & strEventLogName & "'")

    For Each objLogfile in colLogFiles
    Dim backupFilename
    backupFilename= "C:\" & strEventLogName & "_" & GetFormattedTimestamp() _
        & ".evt"

        errBackupLog = objLogFile.BackupEventLog(backupFilename)
        If errBackupLog <> 0 Then
            WScript.Echo "The " & strEventLogName & " event log on " _
                & strComputer & " could not be backed up."
        Else
            objLogFile.ClearEventLog()
        End If
    Next
End Sub

Private Function GetFormattedTimestamp()
    Dim timestamp
    timestamp = Now

    GetFormattedTimestamp = Year(timestamp) _
        & LPad(Month(timestamp), 2, "0") _
        & LPad(Day(timestamp), 2, "0") _
        & "_" & Replace(FormatDateTime(timestamp, 4), ":", "")

End Function

Private Function LPad( _
    strValue, _
    nLength, _
    strPadCharacter)

    Dim strPaddedValue

    strPaddedValue = strValue

    While (Len(strPaddedValue) < nLength)
        strPaddedValue = strPadCharacter & strPaddedValue
    WEnd

    LPad = strPaddedValue
End Function
```

Note that you want to ensure you invoke the script using cscript.exe -- not
wscript.exe -- as shown below:

{{< console-block-start >}}

C:\\&gt;{{< kbd `cscript
"\NotBackedUp\Public\Toolbox\Scripts\Clear Event Logs.vbs" ` >}}

{{< sample-block >}}

Microsoft (R) Windows Script Host Version 5.8\
Copyright (C) Microsoft Corporation. All rights reserved.\
\
Clearing event logs on localhost...\
Clearing 'Application' event log on localhost...\
Clearing 'HardwareEvents' event log on localhost...\
Clearing 'Internet Explorer' event log on localhost...\
Clearing 'Key Management Service' event log on localhost...\
Clearing 'OAlerts' event log on localhost...\
Clearing 'Security' event log on localhost...\
Clearing 'System' event log on localhost...\
Clearing 'Windows PowerShell' event log on localhost...\
Done

{{< /sample-block >}}

{{< console-block-end >}}

Also note that it's very easy to clear the event logs on a remote machine
(assuming you have the necessary permissions and firewall ports open), simply by
specifying the server name as a parameter to the script. If it's not readily
apparent from the script above, the event logs are saved to the root of the C:
drive with a corresponding timestamp (for example,
Application_20110301_0559.evt) and subsequently cleared.

It's also probably worth mentioning that the current version of this script
isn't "bulletproof" -- meaning that you may still see a few warnings or errors
in the **Administrative Events** view of the Event Viewer MMC snap-in. This is
because event logs nested under **Applications and Services Logs** (such as
**TerminalServices-PNPDevices**) are not currently detected (and therefore
subsequently saved/cleared). Honestly, this has never been enough of a pain for
me to actually invest the effort in fixing the script.
