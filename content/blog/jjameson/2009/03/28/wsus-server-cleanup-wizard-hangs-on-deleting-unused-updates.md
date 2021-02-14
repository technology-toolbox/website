---
title: "WSUS Server Cleanup Wizard Hangs on \"Deleting unused updates...\""
date: 2009-03-28T01:20:00+08:00
excerpt: "While cleaning off my Desktop this morning, I came across a file that I created back in December capturing my notes from a problem I was having with Windows Server Update Services (WSUS). Evidently I intended to blog about the issue, but apparently this..."
draft: true
categories: ["Infrastructure"]
tags: ["WSUS", "Infrastructure"]
---

> **Note**
> 
>       This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/28/wsus-server-cleanup-wizard-hangs-on-deleting-unused-updates.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/28/wsus-server-cleanup-wizard-hangs-on-deleting-unused-updates.aspx)
> 
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that
> blog ever goes away.

While cleaning off my Desktop this morning, I came across a file that I created
back in December capturing my notes from a problem I was having with Windows
Server Update Services (WSUS). Evidently I intended to blog about the issue,
but apparently this task kept getting pushed to the back burner. Before moving
on to more fun things today, I thought I should get this off my to-do list.

Back in December, I noticed that my local WSUS server showed that I had
*24,890* updates downloaded and available for install. I remember thinking
something to the effect of: "Wow, that's a lot of updates!"

Looking at my **C:\WSUS** folder (where the updates are downloaded),
I found that it contained the following:

- 51.4 GB of content
- 6,509 files in 258 folders

I then ran the WSUS **Server Cleanup Wizard** as I had attempted
(unsuccessfully) on several occasions in the past, selecting all of the options:

- **Unused updates and update revisions**
- **Computers not contacting the server**
- **Unneeded update files**
- **Expired updates**
- **Superseded updates**

As with my previous attempts, the cleanup appeared to hang on the following
step:

> Deleting unused updates...

I decided that I should try to show a little more patience than I had in
the past and consequently let it run for over 24 hours (thinking it might eventually
finish). You can imagine my frustration when I came back the next morning only
to find that it was still <q>"Deleting unused updates..."</q>

I then tried running the WsusDBMaintenance SQL script -- hoping that might
somehow help the situation. Unfortunately, when I ran the **Server Cleanup
Wizard** again, I got the same result (i.e. it hung on the <q>"Deleting
unused updates..."</q> step).

I then ran the **Server Cleanup Wizard** once again, but this
time only selecting the first option (**Unused updates and update revisions**).
While it was still very, very slow on the <q>"Deleting unused updates..."</q>
step, I was able to verify (via the progress bar) that it was in fact progressing.
I let it run to completion and observed the following results when it was finished:

> Unused updates deleted 3082
> 
> Unused update revisions deleted: 5448

I then ran the **Server Cleanup Wizard** again, but this time
only selecting the third option (**Unneeded update files**). When
that was finished, I observed the following:

> Disk space freed by deleting unused content files: 15441 MB

I confirmed this by rescanning my **C:\WSUS** folder:

- 36.3 GB of content
- 4,285 files in 258 folders

Since December, I have been able to run the **Server Cleanup Wizard**
repeatedly with all of the options selected, and it usually completes in a matter
of minutes. For some reason, it was only that first time through that I needed
to run the steps individually to avoid the hang. Go figure.

