---
title: "Resolving Issues After Applying Hyper-V Snapshot"
date: 2011-03-11T21:04:00+08:00
excerpt: "This morning I rolled back one of my development VMs to a snapshot I created about a month ago. When I subsequently tried to login with my domain credentials, I encountered the following error: 
 The trust relationship between this workstation and the..."
draft: true
categories: ["Infrastructure"]
tags: ["Windows Server", "Infrastructure", "Virtualization"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2011/03/12/resolving-issues-after-applying-hyper-v-snapshot.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/03/12/resolving-issues-after-applying-hyper-v-snapshot.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

This morning I rolled back one of my development VMs to a snapshot I created about a month ago. When I subsequently tried to login with my domain credentials, I encountered the following error:

> The trust relationship between this workstation and the primary domain failed.

I did a quick Internet search for this error and found the following KB article:

<cite>Trust Relationship Between Workstation and Domain Fails</cite>
[http://support.microsoft.com/kb/162797](http://support.microsoft.com/kb/162797)

While you *could* follow the instructions in KB 162797 to resolve this error (removing the computer from the domain and then adding it back), there's a much easier way to resolve the error:

<cite>How to use Netdom.exe to reset machine account passwords of a Windows
Server domain controller</cite>
[http://support.microsoft.com/kb/325850](http://support.microsoft.com/kb/325850)

Don't be put off by the title of this KB article. You just need to read a little bit into it:

> [...] This procedure is most frequently used on domain controllers, but also
> applies to any Windows machine account.

To resolve the error after applying an old Hyper-V snapshot on a VM joined to a domain:

1. Login to the VM using a local administrator account.

2. Open an administrator command prompt and run the following command:
   
   ```
   netdom resetpwd /s:{server} /ud:{DOMAIN\user} /pd:*
   ```

For example:

    ```
    netdom resetpwd /s:XAVIER1 /ud:TECHTOOLBOX\jjameson /pd:*
    ```

> **Note**
> 
>       XAVIER1 is one of the domain controllers in my home lab (TECHTOOLBOX).

3. Logout and log back in using a domain account.

Note that this issue doesn't always occur when rolling back a snapshot. It depends on how old the snapshot is (specifically whether or not the machine account password has changed in the domain since the snapshot was taken).

One of the other things I've learned about using snapshots with a domain-joined VM is that you should be sure to enable the **Time synchronization
**service on the VM. I typically disable this service on domain-joined VMs (since the time is synchronized from the domain controller). However, when using snapshots, the latency in waiting for the time to synchronize after applying a snapshot can quickly become unbearable (especially if you are frequently applying a snapshot).

