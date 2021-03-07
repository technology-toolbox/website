---
title: "Using SysPrep'ed VHDs for New Hyper-V Virtual Machines"
date: 2009-08-13T10:16:00-06:00
excerpt: "As noted in my previous post , I spent 7 days in \"alpha\" training last week for SharePoint 2010. 
 Consequently, one of my goals for this week was to update my \"dogfood\" VM with the CTP build of SharePoint Server 2010, so that I can continue building..."
aliases: ["/blog/jjameson/archive/2009/08/12/using-sysprep-ed-vhds-for-new-hyper-v-virtual-machines.aspx", "/blog/jjameson/archive/2009/08/13/using-sysprep-ed-vhds-for-new-hyper-v-virtual-machines.aspx"]
draft: true
categories: ["My System", "Development", "Infrastructure"]
tags: ["My System", "Core Development", "Virtualization"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/08/13/using-sysprep-ed-vhds-for-new-hyper-v-virtual-machines.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/08/13/using-sysprep-ed-vhds-for-new-hyper-v-virtual-machines.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

As noted in my [previous post](/blog/jjameson/2009/08/13/sharepoint-2010-sneak-peek), I spent 7 days in "alpha" training last week for SharePoint 2010.

Consequently, one of my goals for this week was to update my "dogfood" VM with the CTP build of SharePoint Server 2010, so that I can continue building on the foundation of what I learned last week and dive deeper into the new features of the product.

This is the quintessential scenario for why I love virtual machines. I can quickly "spin up" a new VM and then proceed to load various products such as SQL Server 2008, SharePoint Server 2010, and Visual Studio 2010; and, since it's isolated in a VM, there's no risk with exploring early, pre-release builds of software.

Note that I don't start from scratch each time I need to create a new VM. Rather, I start from a SysPrep'ed image of the particular operating system that I want to use.

In this case, I copied my ws2008-std-x64.vhd (Windows Server 2008 Standard x64 with Service Pack 2) over to the folder for my DOGFOOD VM and renamed it -- yep, you guessed it -- dogfood.vhd.

I then proceeded to start the DOGFOOD VM using Hyper-V Manager. Unfortunately, I encountered the following error:

{{< blockquote "font-italic text-danger" >}}

Virtual Machine Connection

The application encountered an error while attempting to change the state of 'dogfood'

'dogfood' failed to start.

Microsoft Emulated IDE Controller (Instance ID {GUID}): Failed to power on with Error 'General access denied error'

IDE/ATAPI: Could not attach 'C:\NotBackedUp\VMs\dogfood\dogfood.vhd' to location 0/0 of IDE Controller. Error: 'General access denied error'

The file 'C:\NotBackedUp\VMs\dogfood\dogfood.vhd' does not have the required security settings. Error: 'General access denied error'

{{< /blockquote >}}

Using the cacls.exe utility, I examined the permissions specified for a VHD used by one of the other VMs that was already running on the same server. I found the following:

- NT VIRTUAL MACHINE\48FEF7FD-04D0-4AFF-B6F F-4F0D159DC40E:(special access:)
  - READ\_CONTROL
  - SYNCHRONIZE
  - FILE\_GENERIC\_READ
  - FILE\_GENERIC\_WRITE
  - FILE\_READ\_DATA
  - FILE\_WRITE\_DATA
  - FILE\_APPEND\_DATA
  - FILE\_READ\_EA
  - FILE\_WRITE\_EA
  - FILE\_READ\_ATTRIBUTES
  - FILE\_WRITE\_ATTRIBUTES
- NT AUTHORITY\SYSTEM:(ID)F
- BUILTIN\Administrators:(ID)F
- BUILTIN\Users:(ID)R

Note that the list of discrete permissions listed for the NT VIRTUAL MACHINE service account essentially equates to Read and Write, as observed by running the new icacls.exe tool introduced in Windows Server 2003 SP2, and also included in Windows Vista and Windows Server 2008.

Consequently, you can specify a command like the following to grant permissions on the VHD to the service account used by Hyper-V:

{{< console-block-start >}}

icacls dogfood.vhd /grant "NT VIRTUAL MACHINE\C60995EB-8B7D-46DB-BD1E-3638CD1AEC32":(R,W)

{{< console-block-end >}}

Note that the GUID varies with each VM, so you'll need to tweak this accordingly for your environment. To find the GUID to use, examine the **Virtual Machines** folder for the VM.

Also note that you may need to use the VM settings to remove the VHD from the IDE controller and subsequently re-add it (in other words, it looks like Hyper-V does not recognize that the file permissions have been updated in the background).

It takes about 10 minutes to copy the SysPrep'ed VHD, which is obviously much, much faster than the time it takes to install Windows Server 2008 from scratch (especially when you take patching into account). Overall, it takes me about 25-30 minutes to spin up a new Windows Server 2008 VM, which includes joining it to my Active Directory domain and getting all of the latest patches applied using Windows Server Update Services (which, in the ["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) is enforced through Group Policy).

