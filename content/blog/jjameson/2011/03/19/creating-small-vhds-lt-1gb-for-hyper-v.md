---
title: "Creating Small VHDs (< 1GB) for Hyper-V"
date: 2011-03-19T06:37:00-06:00
excerpt: "In my previous post , I explained how I like to create separate VHDs for data and log files in my SharePoint development VMs. However, given the very small amount of content that I typically load into a SharePoint development environment, these VHDs certainly..."
aliases: ["/blog/jjameson/archive/2011/03/18/creating-small-vhds-lt-1gb-for-hyper-v.aspx", "/blog/jjameson/archive/2011/03/19/creating-small-vhds-lt-1gb-for-hyper-v.aspx"]
draft: true
categories: ["My System", "SharePoint", "Infrastructure"]
tags: ["My System", "MOSS 2007", "Infrastructure", "Virtualization", "SharePoint 2010"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/03/19/creating-small-vhds-lt-1gb-for-hyper-v.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/03/19/creating-small-vhds-lt-1gb-for-hyper-v.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

In
[my previous post](/blog/jjameson/2011/03/19/cdl-for-sharepoint-a-k-a-quot-you-can-never-have-too-many-spindles-quot),
I explained how I like to create separate VHDs for data and log files in my
SharePoint development VMs. However, given the very small amount of content that
I typically load into a SharePoint development environment, these VHDs certainly
don't need to be very large.

In the past, I've typically created data and log VHDs using sizes like 5 GB and
1 GB, respectively. However, as I was thinking about this when writing my
previous post, these admittedly arbitrary numbers seem a little ridiciulous. Am
I ever really going to load *5 GB* of sample content into a local development
VM? Not bloody likely.

The more I think about it, a 1 GB VHD for data sounds like it should be plenty.
Even if I did somehow manage to approach that capacity, I could always expand
the VHD to 2 GB (ensuring I delete any snapshots first, of course) -- since this
is now very easy to do in Windows Server 2008. [Unlike back in the days of
Windows Server 2003, with Windows Server 2008 you no longer need a third-party
tool like Partition Magic simply to expand a volume.]

However, using my general rule of thumb that log storage should be roughly 20%
of data storage, this means I should create a 200 MB VHD for the log files. [To
be honest, I don't recall exactly why I use this "20% rule" but it seems to have
served me well over the years.]

Besides, if you configure all databases in your development environment to use
the simple recovery model, then your transaction logs should never get very
large to begin with -- so 200 MB should be plenty, right?

However, there's a problem.

When creating a new VHD using Hyper-V Manager, the minimum size for a VHD is 1
GB.

This morning, I did a quick search for how to create a VHD using PowerShell and
discovered the following post:

{{< reference
title="Hyper-V WMI Using PowerShell Scripts -- Part 2 (VHD Creation)"
linkHref="http://blogs.msdn.com/b/taylorb/archive/2008/05/02/hyper-v-wmi-using-powershell-scripts-part-2-vhd-creation.aspx" >}}

[If, like me, you haven't come across that blog before, it's definitely worth
taking a look at. Taylor Brown from the Hyper-V product team shares a lot of
great tips and useful scripts.]

Note that Taylor has
[an improved version of his original post](http://blogs.msdn.com/b/taylorb/archive/2008/06/18/hyper-v-wmi-rich-error-messages-for-non-zero-returnvalue-no-more-32773-32768-32700.aspx)
for creating VHDs using PowerShell that decrypts the mystical return codes from
WMI.

In short (and with no error handling), use something like the following to
create a 200 MB VHD:

```
$vhdService = Get-WmiObject -Class "Msvm_ImageManagementService" `
    -namespace "root\virtualization"

$vhdService.CreateDynamicVirtualHardDisk(
    "C:\NotBackedUp\VMs\foobar5\foobar5_Log01.vhd",
    200MB)
```

Thanks, Taylor, for sharing this useful PowerShell script.

> **Update (2011-04-14)**
>
> Depending on the specific service applications that you need to configure in
> your SharePoint 2010 development environment, 200 MB not be sufficient for
> transaction log storage (even if you
> [configure your SharePoint databases to use the Simple recovery model by default](/blog/jjameson/2011/03/19/using-the-simple-recovery-model-for-sharepoint-development-environments)).
> Unfortunately, I discovered that when I tried to configure numerous service
> applications on my development VM (to match my client's Production
> environment) I ran out of space on my L: drive.
>
> Instead of the 200 MB I originally thought that I could get away with, I now
> use 500 MB. So far, I haven't encountered any issues with this increased size.

Note that you'll probably need to change the permissions on the new VHD in order
to avoid an "Access Denied" error message after attaching the VHD to a VM:

{{< console-block-start >}}

icacls foobar5\_Log01.vhd /grant "NT VIRTUAL MACHINE\{GUID}":(R,W)

{{< console-block-end >}}

This is described in more detail in
[one of my previous posts](/blog/jjameson/2009/08/13/using-sysprep-ed-vhds-for-new-hyper-v-virtual-machines).

