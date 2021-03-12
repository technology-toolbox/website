---
title: "Tips & Tricks for Running PowerShell Scripts as Scheduled Tasks"
date: 2011-11-19T02:26:24-07:00
excerpt: "In my previous post, I described the PowerShell script used to rebuild the Development environment for TechnologyToolbox.com on a daily basis. This post explains the subtleties of running the script -- or, more generally, any PowerShell script -- using the Windows Task Scheduler..."
aliases: ["/blog/jjameson/archive/2011/11/18/tips-tricks-for-running-powershell-scripts-as-scheduled-tasks.aspx", "/blog/jjameson/archive/2011/11/19/tips-tricks-for-running-powershell-scripts-as-scheduled-tasks.aspx"]
draft: true
categories: ["Development", "My System"]
tags: ["My System", "PowerShell"]
---

In
[my previous post](/blog/jjameson/2011/11/17/building-technologytoolbox-com-part-8),
I described the PowerShell script used to rebuild the Development environment
for TechnologyToolbox.com on a daily basis. This post explains the subtleties of
running the script -- or, more generally, *any* PowerShell script -- using the
Windows Task Scheduler.

### Understanding the issues

Let's start with a very simple PowerShell script to use as an example
(Temp.ps1):

```
$ErrorActionPreference = "Stop"

Write-Host "Changing to TEMP folder..."
cd $env:TEMP

Write-Host "Creating file..."
New-Item Temp.txt -Type File -Value "foobar"

Write-Host "Copying file..."
Copy-Item Temp.txt Temp-Copy.txt

Write-Host "Success"
```

When you run this script the first time, it should complete successfully and the
output should resemble the following:

```
PS C:\Users\jjameson\AppData\Local\Temp> {{< kbd "C:\Temp.ps1" >}}
Changing to TEMP folder...
Creating file...

    Directory: C:\Users\jjameson\AppData\Local\Temp

Mode                LastWriteTime     Length Name
----                -------------     ------ ----
-a---        11/19/2011   5:37 AM          6 Temp.txt
Copying file...
Success

PS C:\Users\jjameson\AppData\Local\Temp>
```

However, when you run the script again, an error occurs:

```
PS C:\Users\jjameson\AppData\Local\Temp> C:\Temp.ps1
Changing to TEMP folder...
Creating file...
New-Item : The file 'C:\Users\jjameson\AppData\Local\Temp\Temp.txt' already exists.
At C:\Temp.ps1:7 char:9
+ New-Item <<<<  Temp.txt -Type File -Value "foobar"
    + CategoryInfo          : WriteError: (C:\Users\jjames...l\Temp\Temp.txt:String) [New-Item], IOException
    + FullyQualifiedErrorId : NewItemIOError,Microsoft.PowerShell.Commands.NewItemCommand

PS C:\Users\jjameson\AppData\Local\Temp>
```

Now, imagine the PowerScript script actually did something useful and we
configured it to run as a scheduled task with the following properties:

- **Action:** Start a program
- **Program/script:** PowerShell.exe
- **Add arguments:** -Command "C:\Temp.ps1"

Deleting the temporary files and then running the scheduled task results in the
following message in the **Last Run Result** column:

{{< blockquote "font-italic" >}}

The operation completed successfully. (0x0)

{{< /blockquote >}}

If you were to look in the %TEMP% folder, you would see the two text files
created by the script.

However, when you run the scheduled task again (without deleting the files) the
**Last Run Result** column displays:

{{< blockquote "font-italic" >}}

(0x1)

{{< /blockquote >}}

This is good because it tells us something went wrong running the PowerShell
script, but imagine we didn't already know what the underlying error actually
was. Unfortunately, the **History** tab provides no additional information other
than the fact that PowerShell.exe exited with return code 1.

We need to be able to inspect a log file containing all of the output from the
script.

Like me, you might try using the **
[Start-Transcript](http://technet.microsoft.com/en-us/library/dd347721.aspx)**
cmdlet to create a log file:

```
$ErrorActionPreference = "Stop"

Start-Transcript -Path "$env:TEMP\Temp.log"
Write-Host "Changing to TEMP folder..."
...
Write-Host "Success"

Stop-Transcript
```

> **Note**
>
> In order to avoid issues when running the script interactively from a
> PowerShell window, we really should call **Stop-Transcript** when an error
> occurs (i.e. by using **Trap**). Otherwise, if an error occurs while running
> the script from a PowerShell prompt, the transcript file remains open and you
> need to type {{< kbd "Stop-Transcript" >}} to close it.

Running the scheduled task now produces the following in the Temp.log file (as
viewed in Notepad):

{{< log-excerpt >}}

```
**********************
Windows PowerShell Transcript Start
Start time: 20111119062551
Username  : TECHTOOLBOX\jjameson
Machine	  : FOOBAR5 (Microsoft Windows NT 6.1.7601 Service Pack 1)
**********************
Transcript started, output file is C:\Users\jjameson\AppData\Local\Temp\Temp.log
Changing to TEMP folder...Creating file...New-Item : The file 'C:\Users\jjameson\AppData\Local\Temp\Temp.txt' already exists.At C:\Temp.ps1:9 char:9+ New-Item <<<<  Temp.txt -Type File -Value "foobar"    + CategoryInfo          : WriteError: (C:\Users\jjames...l\Temp\Temp.txt:String) [New-Item], IOException    + FullyQualifiedErrorId : NewItemIOError,Microsoft.PowerShell.Commands.NewItemCommand **********************
Windows PowerShell Transcript End
End time: 20111119062551
***********************
```

{{< /log-excerpt >}}

While this resembles the expected output, it's definitely not very easy to read.
This is a known issue:

{{< reference title="How to get Notepad to honor Start-Transcript line breaks"
linkHref="http://social.technet.microsoft.com/Forums/en-US/ITCG/thread/e4784b39-ed97-4c6c-bc82-372be94e8c01/" >}}

Some people suggest either adding "``r`n`" to the end of every `Write-Host`
statement or avoiding `Write-Host` altogether and instead simply pipe a string
to **Out-Default**. The latter option seems awkward, in my opinion, so let's use
the former:

```
$ErrorActionPreference = "Stop"

Start-Transcript -Path "$env:TEMP\Temp.log"

Write-Host "Changing to TEMP folder...`r`n"
cd $env:TEMP

Write-Host "Creating file...`r`n"
New-Item Temp.txt -Type File -Value "foobar"

Write-Host "Copying file...`r`n"
Copy-Item Temp.txt Temp-Copy.txt

Write-Host "Success`r`n"

Stop-Transcript
```

This improves the readability of the log file in Notepad significantly. However,
the exception message still appears on a single line (rather than as five
separate lines and nicely indented like how it appears when running
interactively).

While I could probably live with the exception message formatting issue, there's
a much larger issue with the **Start-Transcript** cmdlet:

{{< reference title="Unable to capture ALL session output into a transcript"
linkHref="https://connect.microsoft.com/PowerShell/feedback/details/315875/unable-to-capture-all-session-output-into-a-transcript" >}}

Once I discovered this, I decided that while **Start-Transcript** seems like a
good idea, it's not quite yet ready for primetime.

Instead of trying to use **Start-Transcript** to create a log file for the
PowerShell script, let's try modifying the scheduled task to create the log file
instead, by changing the arguments of the scheduled task action to the
following:

> -Command "[C:\Temp.ps1](file:///C:/Temp.ps1)" &gt; "%TEMP%\Temp.log"

Deleting the temporary files (to start with the "happy path" scenario first) and
then running the scheduled task results in the following Temp.log file:

{{< log-excerpt >}}

```

    Directory: C:\Users\jjameson\AppData\Local\Temp

Mode                LastWriteTime     Length Name
----                -------------     ------ ----
-a---        11/19/2011   8:19 AM          6 Temp.txt
```

{{< /log-excerpt >}}

Notice the log file doesn't show the `Write-Host` output (e.g. "{{<
sample-output "Changing to TEMP folder..." >}}") -- which would make it much
more difficult to troubleshoot an error (or bug) in a lengthy script.

### Use a command file to start PowerShell script and create log file

To circumvent these issues, I recommend creating a simple command file that
invokes the PowerShell script and logs the output (including errors) to a file.
Then create a scheduled task which runs the command file.

Here is the sample command file from my previous post (Rebuild Website.cmd):

```
PowerShell.exe -Command ".\'Rebuild Website.ps1'; Exit $LASTEXITCODE" > "Rebuild Website.log" 2>&1
EXIT %ERRORLEVEL%
```

Notice the use of `$LASTEXITCODE` and `EXIT %ERRORLEVEL%` in order to "bubble
up" any non-zero return code from PowerShell to the **Last Run Result** column
in Task Scheduler. In other words, when an error occurs while running the
PowerShell script, we don't want the scheduled task to report "{{< sample-output
"The operation completed successfully. (0x0)" >}}"; rather it should indicate
that something bad happpened (which would trigger us to examine the
corresponding log file to investigate the issue).

I also use "`2>&1`" to redirect `stderr` to `stdout` to ensure error messages
are written to the log file as well as the normal output. This is a trick I
covered in
[a previous blog post](/blog/jjameson/2009/03/27/redirecting-stderr-to-stdout).

> **Important**
>
> Depending on the content of the PowerShell script, you may encounter issues
> when redirecting `stderr` to `stdout`. For example, I originally used RoboCopy
> in the PowerShell script described in my previous post (to copy files from the
> Release server to the Web server). Consequently, I encountered a bug in
> PowerShell that is described in the following blog post:
>
> {{< reference
> title="Workaround: The OS handle's position is not what FileStream expected"
> linkHref="http://www.leeholmes.com/blog/2008/07/30/workaround-the-os-handles-position-is-not-what-filestream-expected/"
>
> > }}
>
> To avoid this bug, I replaced the use of RoboCopy with `Copy-Item`.

The properties for the corresponding scheduled task are as follows:

- **General**
  - **Name:** Rebuild website - www-dev.technologytoolbox.com
  - **When running the task, use the following user account:** SYSTEM
  - **Run with highest privileges:** (checked)
- **Actions**
  - **Start a program**
    - **Program/script:** "C:\NotBackedUp\TechnologyToolbox\Caelum\Main\Source\Deployment Files\Scripts\Rebuild Website.cmd"
    - **Start in:** C:\NotBackedUp\TechnologyToolbox\Caelum\Main\Source\Deployment Files\Scripts

Here is a sample log file (as viewed in Notepad), which shows the `Write-Host`
messages as well as other output (e.g. "{{< sample-output
"processed file: C:\inetpub\wwwroot\..." >}}" from icacls.exe):

{{< log-excerpt >}}

```
Defaulting to latest build for Caelum...
Defaulting to latest build for Subtext...
Defaulting to Debug build configuration...
Stopping website (www-dev.technologytoolbox.com)...
Successfully stopped website (www-dev.technologytoolbox.com).
Removing website (www-dev.technologytoolbox.com)...
Successfully removed website (www-dev.technologytoolbox.com).
Removing website folder (C:\inetpub\wwwroot\www-dev.technologytoolbox.com)...
Successfully removed website folder (C:\inetpub\wwwroot\www-dev.technologytoolbox.com).
Removing application pool (www-dev.technologytoolbox.com)...
Successfully removed application pool (www-dev.technologytoolbox.com).
Creating application pool (www-dev.technologytoolbox.com)...
Successfully created application pool (www-dev.technologytoolbox.com).
Creating website folder (C:\inetpub\wwwroot\www-dev.technologytoolbox.com)...
Successfully created website folder (C:\inetpub\wwwroot\www-dev.technologytoolbox.com).
Creating website (C:\inetpub\wwwroot\www-dev.technologytoolbox.com)...
Successfully created website (C:\inetpub\wwwroot\www-dev.technologytoolbox.com).
Copying Subtext website content...
Successfully copied Subtext website content.
Granting read/write access on Subtext App_Data folder...
processed file: C:\inetpub\wwwroot\www-dev.technologytoolbox.com\blog\App_Data
Successfully processed 1 files; Failed processing 0 files
Successfully granted read/write access on Subtext App_Data folder.
Creating new application for blog site...
Successfully created new application for blog site.
Copying Caelum website content...
Successfully copied Caelum website content.
Successfully rebuilt Web application.
```

{{< /log-excerpt >}}

