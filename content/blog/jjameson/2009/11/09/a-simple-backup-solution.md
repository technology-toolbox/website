---
title: "A Simple Backup Solution"
date: 2009-11-09T05:44:00-07:00
excerpt: "As I've mentioned before, I don't spend much money or time maintaining the \"Jameson Datacenter\" (a.k.a. my home lab). However, that doesn't mean that I treat my infrastructure lightly. 
 In previous posts, I've covered many of the Group Policy objects..."
aliases: ["/blog/jjameson/archive/2009/11/08/a-simple-backup-solution.aspx", "/blog/jjameson/archive/2009/11/09/a-simple-backup-solution.aspx"]
draft: true
categories: ["My System", "Infrastructure"]
tags: ["My System", "Windows Server", "Infrastructure"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/11/09/a-simple-backup-solution.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/11/09/a-simple-backup-solution.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

As I've mentioned before, I don't spend much money or time maintaining the
["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) (a.k.a.
my home lab). However, that doesn't mean that I treat my infrastructure lightly.

In previous posts, I've covered many of the Group Policy objects that I use to
minimize the maintenance effort associated with running more than a dozen
servers (mostly virtual). In this post, I'll provide the details on how I backup
these servers.

I should preface this by saying this is not meant to be an "enterprise-level"
backup solution. Rather it is simply meant to provide a cheap (actually free)
and easy solution to the problem of ensuring you can recover from data loss.
Note that data loss rarely occurs through some sort of hardware failure or Act
of God (as the insurance folks like to put it). Rather the majority of the time
someone accidentally overwrites or deletes a file -- or, gasp, a complete folder
hierarchy -- and you subsequently need to restore the data from a backup.

For as long as I can remember, Windows Server has included the NTBackup utility.
I'm guessing from the name that this has been around since the days of Windows
NT 3.1, but honestly I don't believe I even started running Windows NT until
version 3.5. Or was it 3.51? I can't remember. Anyway, I certainly haven't been
running NTBackup since then.

Here is the simple batch file that I use to perform scheduled backups:

```
@echo off

setlocal

set BACKUP_TYPE=normal

if ("%1") NEQ ("") set BACKUP_TYPE=%1

for /f "tokens=2-4 delims=/ " %%i in ('date /t') do set currentDate=%%k-%%i-%%j
for /f "tokens=1-2" %%i in ('time /t') do set currentTime=%%i %%j
set BACKUP_TIMESTAMP=%currentDate%-%currentTime:~0,2%-%currentTime:~3,2%-%currentTime:~6,2%

set BACKUP_FILE=D:\NotBackedUp\Backups\Backup-%BACKUP_TYPE%-%BACKUP_TIMESTAMP%.bkf

:: ----------------------------------------------------------------------------
call :LogMessage "Starting backup..."
call :LogMessage "BACKUP_TYPE: %BACKUP_TYPE%"
call :LogMessage "BACKUP_FILE: %BACKUP_FILE%"

C:\WINDOWS\system32\ntbackup.exe backup C:\BackedUp /n "Backup created %BACKUP_TIMESTAMP%" /m %BACKUP_TYPE% /j "Backup (%BACKUP_TYPE%)" /f "%BACKUP_FILE%"
if %ERRORLEVEL% neq 0 goto Errors

call :LogMessage "Successfully completed backup."

goto :eof

:: ----------------------------------------------------------------------------
::
:LogMessage

REM Strip leading and trailing quotes and then display message with timestamp
set MESSAGE=%1
set MESSAGE=%MESSAGE:~1,-1%

for /f "tokens=2-4 delims=/ " %%i in ('date /t') do set currentDate=%%k-%%i-%%j
for /f "tokens=1-2" %%i in ('time /t') do set currentTime=%%i %%j
echo %currentDate% %currentTime% - %MESSAGE%

goto :eof

:: ----------------------------------------------------------------------------
::
:Errors

echo Warning! One or more errors detected.
```

If you've seen any of my scripts before, then you'll quickly notice the typical
`LogMessage` "function" that I use to write messages prefixed with a timestamp.
For example here's the output from the log for this morning's backup:

{{< log-excerpt >}}

```
2009-11-09 12:30 AM - Starting backup...
2009-11-09 12:30 AM - BACKUP_TYPE: differential
2009-11-09 12:30 AM - BACKUP_FILE: D:\NotBackedUp\Backups\Backup-differential-2009-11-09-12-30-AM.bkf
2009-11-09 12:31 AM - Successfully completed backup.
```

{{< /log-excerpt >}}

I use similar token parsing of the output from the **
[date](http://technet.microsoft.com/en-us/library/cc732776%28WS.10%29.aspx)**
and
[**time**](http://technet.microsoft.com/en-us/library/cc770579%28WS.10%29.aspx)
system commands to generate the name of the backup file (e.g.
{{< sample-output "Backup-differential-2009-11-09-12-30-AM.bkf" >}}).

Also note that the
[type of backup](http://technet.microsoft.com/en-us/library/cc784306%28WS.10%29.aspx)
(e.g. **normal** or **differential**) can be specified as a parameter when
running the batch file. This is really powerful for scheduling different types
of backups on various schedules.

Here are the scheduled backups on one of my servers (BEAST):

{{< table class="small" caption="Scheduled Backups on BEAST" >}}

| Name | Schedule |
| --- | --- |
| Daily Backup | At 12:00 PM every day |
| Differential Backup | At 12:30 AM every day |
| Full Backup | At 1:00 AM every Sun of every week |

{{< /table >}}

The **Daily Backup** task is configured as follows:

- **Run:** C:\BackedUp\Backup.cmd daily &gt;&gt; Backup.log
- **Start in:** C:\BackedUp
- **Run as:** TECHTOOLBOX\svc-backup

Note that I specifically chose the middle of the day to perform daily backups so
that I could potentially recover a file that was created in the morning but
mistakenly deleted in the afternoon. I suppose I could schedule incremental
backups throughout the day, but honestly, I haven't seen the need given my
situation.

Also note that the service account that I use for backups
(TECHTOOLBOX\svc-backup) is only a member of the **Backup Operators** group. It
is not a member of the **Administrators** group.

Consequently there's a known issue with running batch files using scheduled
tasks due to out-of-the-box security restrictions on cmd.exe:

{{< reference title="\"Access is denied\" error message when you run a batch job on a Windows Server 2003-based computer" linkHref="http://support.microsoft.com/kb/867466" >}}

Lastly, note that I am doing a simple disk-to-disk backup on my servers, so if
there's a fire in the Jameson Datacenter (i.e. my basement) and I lose these
servers completely then I'm going to be "hurtin' for certain." However should
there ever be a fire in my basement (Heaven forbid), I'm going to be worried
about a lot more than just restoring my data from backup. Note that I keep
copies of the *really* important stuff (e.g. digital photos and home videos of
my family) on DVDs at my parents' house.

I've read that there's a new backup tool in Windows Server 2008, so I suppose
one of these days I'll need to get around to upgrading my backup solution ;-)

