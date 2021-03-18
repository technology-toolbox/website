---
title: Script to Save Event Logs
date: 2011-03-04T05:35:00-07:00
excerpt:
  "Earlier this week , I shared a script that I frequently use in my development
  environments to clear the event logs (for example, whenever I want to verify
  that one of my VMs \"boots clean\" -- meaning without any errors or warnings).
  Note that prior to..."
aliases:
  [
    "/blog/jjameson/archive/2011/03/03/script-to-save-event-logs.aspx",
    "/blog/jjameson/archive/2011/03/04/script-to-save-event-logs.aspx",
  ]
draft: true
categories: ["Infrastructure", "My System"]
tags: ["Windows Server", "Toolbox"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2011/03/04/script-to-save-event-logs.aspx"
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/03/04/script-to-save-event-logs.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/03/04/script-to-save-event-logs.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

[Earlier this week](/blog/jjameson/2011/03/01/script-to-clear-and-save-event-logs),
I shared a script that I frequently use in my development environments to clear
the event logs (for example, whenever I want to verify that one of my VMs "boots
clean" -- meaning without any errors or warnings). Note that prior to clearing
each of the event logs, the script first saves a copy (to C:\ with a timestamp
in the filename) just in case I need to go back and look at them.

While you could easily modify the original script I provided in order to save --
but not clear -- the event logs, as I was writing
[my previous post this morning](/blog/jjameson/2011/03/04/identifying-logon-failures-on-a-web-site),
I thought it would be helpful to share a different script in which I have
already done just that.

Here is the script that I occasionally use whenever I need to analyze event logs
from a Production environment. I typically ask one of the members of the
Operations team to run the script for me (for each of the servers that I need to
analyze) and subsequently copy the saved copies of the event logs to some
location that I actually have access to. [I don't typically have -- nor want --
access to the Production environments on projects I'm involved with.]

Note that I am typically only interested in the Application and System logs. If
you want to save copies of other event logs, you'll need to tweak the script
below.

### Save Event Logs.vbs

```
If WScript.Arguments.Count > 1 Then
    WScript.Echo
    WScript.Echo "Usage: cscript ""Save Event Logs.vbs"" [computer name]"
    WScript.Echo
    WScript.Quit
End If

Dim strComputer ' As String

If WScript.Arguments.Count > 0 Then
    strComputer= WScript.Arguments(0)
Else
    strComputer= "localhost"
End If

SaveEventLogs strComputer

WScript.Echo "Done"

Private Sub SaveEventLogs(strComputer)
    WScript.Echo "Saving event logs on " & strComputer & "..."

    SaveEventLog strComputer, "Application"
    'SaveEventLog strComputer, "Security"
    SaveEventLog strComputer, "System"
End Sub

Private Sub SaveEventLog(strComputer, strEventLogName)
    Set objWMIService = GetObject("winmgmts:" _
        & "{impersonationLevel=impersonate,(Backup)}!\\" & _
            strComputer & "\root\cimv2")

    Set colLogFiles = objWMIService.ExecQuery _
        ("Select * from Win32_NTEventLogFile where LogFileName='" _
            & strEventLogName & "'")

    For Each objLogfile in colLogFiles
        Dim backupFilename
        backupFilename = "\"

        If (Not strComputer = "localhost") Then
            backupFilename = backupFilename & strComputer & "_"
        End If

        backupFilename = backupFilename & strEventLogName & "_" _
            & GetFormattedTimestamp() & ".evt"

        errBackupLog = objLogFile.BackupEventLog(backupFilename)
        If errBackupLog <> 0 Then
            WScript.Echo "The " & strEventLogName & " event log on " _
                & strComputer & " could not be backed up."
        End If
    Next
End Sub

Private Function GetFormattedTimestamp
    Dim timestamp
    timestamp = Now

    GetFormattedTimestamp = Year(timestamp) _
        & LPad(Month(timestamp), 2, "0") _
        & LPad(Day(timestamp), 2, "0") _
        & "_" & Replace(FormatDateTime(timestamp, 4),":","")

End Function

Private Function LPad(strValue, nLength, strPadCharacter)
    Dim strPaddedValue

    strPaddedValue = strValue

    While (Len(strPaddedValue) < nLength)
        strPaddedValue = strPadCharacter & strPaddedValue
    WEnd

    LPad = strPaddedValue
End Function
```
