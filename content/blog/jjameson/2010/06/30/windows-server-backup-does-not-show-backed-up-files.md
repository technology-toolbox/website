---
title: "Windows Server Backup Does Not Show Backed Up Files"
date: 2010-06-30T14:50:00-06:00
excerpt: "[Sorry the blog has been relatively silent this month -- but, on the other hand, I did manage to make time for a vacation to Arizona this month (Sedona, Grand Canyon, and then Scottsdale). It was a little hot, but a wonderful trip nonetheless.] 
 Here..."
aliases: ["/blog/jjameson/archive/2010/06/30/windows-server-backup-does-not-show-backed-up-files.aspx"]
draft: true
categories: ["Infrastructure"]
tags: ["Windows Server", "Infrastructure"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/06/30/windows-server-backup-does-not-show-backed-up-files.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/06/30/windows-server-backup-does-not-show-backed-up-files.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

[Sorry the blog has been relatively silent this month -- but, on the other hand,         I did manage to make time for a vacation to Arizona this month (Sedona, Grand Canyon,         and then Scottsdale). It was a little hot, but a wonderful trip nonetheless.]

Here's an email that I sent last month regarding an issue I discovered after upgrading         one of my servers to Windows Server 2008 R2:

> ***
>
> 
> **From:** Jeremy Jameson
> **Sent:** Friday, May 21, 2010 11:00 AM
> **To:** [...]
> **Subject:** Windows Server Backup: "Select Items to Recover" does             not show backed up files
>
> On a server running Windows Server 2008 R2, I have completed a backup with the following             options:
>
> - Backup items: Selected files (C:\BackedUp\)
> - Files excluded: None
> - Advanced option: VSS Copy Backup
> - Backup destinations: Local Disk (D:) -- using the **Back up to a volume**
>   option
>
> According to the backup log file, all of the files in the specified location are             successfully backed up:
>
> Backed up C:\
> Backed up C:\BackedUp\
> ...
> Backed up C:\BackedUp\Profiles\
> Backed up C:\BackedUp\Profiles\jjameson\
> Backed up C:\BackedUp\Profiles\jjameson\NTUSER.DAT
> Backed up C:\BackedUp\Profiles\jjameson\ntuser.dat.LOG
> Backed up C:\BackedUp\Profiles\jjameson\ntuser.ini
> Backed up C:\BackedUp\Profiles\jjameson\ntuser.pol
> Backed up C:\BackedUp\Profiles\jjameson\Application Data\
> Backed up C:\BackedUp\Profiles\jjameson\Application Data\Microsoft\
> Backed up C:\BackedUp\Profiles\jjameson\Application Data\Microsoft\CLR Security             Config\
> ...
>
> However, when I try to recover the files, it appears as if the files were not backed             up, as shown in the screenshot below.
>
> {{< figure src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Windows-Server-Backup-Recovery-Wizard-Bug-600x465.png" alt="Windows Server Backup-Recovery Wizard - bug" height="465" width="600" >}}
>
> [(See full-sized image)](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Windows-Server-Backup-Recovery-Wizard-Bug-756x586.png)
>
> Note that the Recovery Wizard doesn't show any files or folders under the C:\BackedUp\Profiles\jjameson             folder (even though the backup log lists the files and folders in this location).
>
> Also note that I am using my **TECHTOOLBOX\jjameson-admin** account             -- which is a Domain Admin account -- to perform the backup and restore operations.             I also added it to the **Backup Operators** group on the server (even             though this seems superfluous).
>
> The permissions on the C:\BackedUp\Profiles\jjameson folder are as follows:
>
> - SYSTEM -- Full Control
> - TECHTOOLBOX\jjameson -- Full Control (note that this is not my TECHTOOLBOX\jjameson-admin
>   account)
>
> It seems like the Recovery Wizard doesn't honor the "security override" feature             of the **Backup Operators** group. I don't recall having any issues             with these permissions when this server was running Windows Server 2003 and I was             using NTBackup.
>
> Doesn't this seem like a bug (since the files can be backed up, but they cannot             be recovered)?
>
> Jeremy Jameson
> **Microsoft Consulting Services**
> ...
>
> ***


Unfortunately, I never received a single response to my inquiry. Fortunately, my         initial suspicion was correct and I was able to resolve the issue a few days later         with a minor change to the NTFS permissions.

Note that in a previous blog post, I described [how I (used to) backup files in the "Jameson Datacenter"](/blog/jjameson/2009/11/09/a-simple-backup-solution) (a.k.a. my home         lab). However, I had to abandon that approach after upgrading to Windows Server         2008 (since NTBackup is no longer supported for backing up files). [Note that although         NTBackup isn't included in Windows Server 2008, you can download it separately,         but the new version only supports restore operations (not backup operations).]

Also note that the Windows Server Backup feature is substantially improved in Windows         Server 2008 R2 -- which is actually one of the reasons why I put off upgrading my         server (BEAST) for so long. If, like me, you tried using Windows Server Backup in         Windows Server 2008 but found it to be severely lacking in functionality, I encourage         you to take a look at the improvements made in Windows Server 2008 R2. While I still         miss NTBackup (mostly because I was very familiar with it), I'm starting to like         Windows Server Backup more and more as time goes on.

So, anyway, back to the bug (at least what I consider to be a bug)...

In order to resolve the issue of backed up files not being displayed in the Recovery         Wizard, all I had to do was grant the local **Administrators** group         access to the files. In other words, the permissions on the C:\BackedUp\Profiles\jjameson         folder are now as follows:

- SYSTEM -- Full Control
- TECHTOOLBOX\jjameson -- Full Control
- Administrators -- Full Control

Once these permission changes were applied, I performed another backup and verified         that the backed up files appeared in the Recovery Wizard as expected.

