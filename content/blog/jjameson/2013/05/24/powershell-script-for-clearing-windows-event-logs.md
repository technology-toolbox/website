---
title: PowerShell Script for Clearing Windows Event Logs
date: 2013-05-24T23:01:34-06:00
excerpt:
  "Sure, you can just type \"wevtutil el | % { wevtutil cl $_ }\" whenever you
  feel like it, but how fun is that?!"
aliases:
  [
    "/blog/jjameson/archive/2013/05/24/powershell-script-for-clearing-windows-event-logs.aspx",
  ]
draft: true
categories: ["Development", "My System"]
tags: ["Core Development", "My System", "PowerShell", "Toolbox", "Windows Server"]
---

A couple of years ago, I shared the
[script I created to quickly clear the event logs on a server](/blog/jjameson/2011/03/01/script-to-clear-and-save-event-logs)
(but saving them first -- just in case I need to go back and retrieve something
from the "archive").

While that script (Clear Event Logs.vbs) is still in my Toolbox, I can't
remember the last time I used it. This is largely due to the fact that I
primarily use **Server Manager** → **Diagnostics** → **Event Viewer** → **Custom
Views** → **Administrative Events** to quickly check on the health of my
development VMs, and with the introduction of the **Applications and Services
Logs** in Windows Server 2008, running that script doesn't always clear
everything in that view.

For example, after running:

```
cscript "Clear Event Logs.vbs"
```

...I would still see lots of warnings in the **Administrative Events** view from
**TerminalServices-PnPDevices**.

Consequently I switched over to using PowerShell to clear *all* the event logs
-- not just the "classic" event logs (i.e. Application, Security, and System).

I started out by simply typing the following in a PowerShell window:

```
wevtutil el | % { wevtutil cl $_ }
```

However, typing that command even once every few days quickly grew tiresome, so
I wrapped it up in a script in my Toolbox. Now I simply run the following to
clear the event logs on a development VM:

{{< console-block-start >}}

C:\NotBackedUp\Public\Toolbox\PowerShell\Clear-EventLogs.ps1

{{< console-block-end >}}

{{< div-block "note important" >}}

> **Important**
>
> Unlike "Clear Event Logs.vbs", Clear-EventLogs.ps1 does not save the event
> logs before clearing them. If you desire to maintain that functionality, then
> you'll need to modify the script accordingly.
>
> Personally, I haven't found the need for doing that since I only clear the
> event logs on development VMs (System Center Operations Manager "barks at you"
> when you clear the event logs on servers that are being monitored -- e.g. the
> Production environment at Technology Toolbox).

{{< /div-block >}}

{{< div-block "note" >}}

> **Note**
>
> You can download this script from my Toolbox repository on GitHub:
>
> [https://github.com/jeremy-jameson/Toolbox](https://github.com/jeremy-jameson/Toolbox)

{{< /div-block >}}

### Clear-EventLogs.ps1

```
<#
.SYNOPSIS
Clears all Windows event logs.

.DESCRIPTION
Clearing all of the Windows event logs is useful in development environments
(for example, to check if a computer reboots "cleanly" -- i.e. without any
errors).

.EXAMPLE
.\Clear-EventLogs.ps1

.NOTES
This script must be run with administrator privileges.
#>

Set-StrictMode -Version Latest
$ErrorActionPreference = "Stop"

wevtutil el | ForEach-Object { wevtutil cl $_ }
```
