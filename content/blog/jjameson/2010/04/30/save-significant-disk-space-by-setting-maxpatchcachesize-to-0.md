---
title: "Save Significant Disk Space by Setting MaxPatchCacheSize to 0"
date: 2010-04-30T14:51:00-06:00
excerpt:
  "A little over two years ago, I wrote a post about installing Visual Studio
  2005 Service Pack 1 , in which I mentioned setting the MaxPatchCacheSize
  registry setting to 0 (in order to save some significant disk space while
  installing the service pack)..."
aliases: ["/blog/jjameson/archive/2010/04/30/save-significant-disk-space-by-setting-maxpatchcachesize-to-0.aspx"]
draft: true
categories: ["My System", "Infrastructure", "Development"]
tags: ["My System", "SQL Server", "Windows Server", "Infrastructure", "Virtualization", "Visual Studio"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/30/save-significant-disk-space-by-setting-maxpatchcachesize-to-0.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/30/save-significant-disk-space-by-setting-maxpatchcachesize-to-0.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

A little over two years ago, I wrote a post about
[installing Visual Studio 2005 Service Pack 1](/blog/jjameson/2008/02/08/installing-visual-studio-2005-sp1),
in which I mentioned setting the MaxPatchCacheSize registry setting to 0 (in
order to save some significant disk space while installing the service pack).
[Note that the credit for this trick really goes to [Heath Stewart](http://blogs.msdn.com/heaths/)
-- as I mentioned in my original post.]

I've also mentioned in various posts that I make heavy use of virtualization in
the "[Jameson Datacenter](/blog/jjameson/2009/09/14/the-jameson-datacenter)"
(a.k.a. my home lab) and that I prefer to keep my VHDs reasonably small. This is
especially valuable when, for example, I need to copy a VM from one of my home
servers and take it "on the road" with me to a customer site.

As such, one of the first things that I typically do when building out a new VM
is to run the following from a command prompt:

{{< console-block-start >}}

reg add HKLM\Software\Policies\Microsoft\Windows\Installer /v MaxPatchCacheSize
/t REG\_DWORD /d 0 /f

{{< console-block-end >}}

Then I move on to installing products based on the intended purpose of the VM.

Here's an excerpt from the
[MSDN page for MaxPatchCacheSize](http://msdn.microsoft.com/en-us/library/aa369798%28VS.85%29.aspx):

{{< blockquote "font-italic" >}}

If this per-machine system policy is set to a value greater than 0, Windows
Installer saves older versions of files in a cache when a patch is applied to an
application. Caching can increase the performance of future installations that
otherwise need to obtain the old files from a original application source.

...

If the value of the MaxPatchCacheSize policy is set to 0, no additional files
are saved.

{{< /blockquote >}}

To understand the value of setting MaxPatchCacheSize to 0 on a VM, take a look
at the following figure, which shows the disk space usage on a freshly built
Windows Server 2008 R2 VM, after installing SQL Server 2008 and subsequently
running Windows Update to install all of the latest patches (including SQL
Server 2008 Service Pack 1).

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-Usage-WS2008-R2-(with-SQL-2008-SP1)-600x426.png"
alt="Disk usage on a Windows Server 2008 R2 VM with SQL Server 2008 SP1 (MaxPatchCacheSize not set)"
class="screenshot" height="426" width="600"
title="Figure 1: Disk usage on a Windows Server 2008 R2 VM with SQL Server 2008 SP1 (MaxPatchCacheSize not set)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-Usage-WS2008-R2-%28with-SQL-2008-SP1%29-1024x727.png)

Note that the \Windows\Installer\$PatchCache$ folder is consuming almost 1 GB of
space on the VHD.

I've mentioned before that a gigabyte or two isn't all that much when your
physical hard drive is several hundred gigabytes in size. However, the same
number on a 20 GB VHD is a different matter altogether.

The following figure shows the disk space usage for a similar configuration
(i.e. a freshly built Windows Server 2008 R2 VM with SQL Server 2008 SP1), but
this time with MaxPatchCacheSize set to 0 prior to starting the installation of
SQL Server.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/After-restricting-MaxPatchCacheSize-600x493.png"
alt="Disk usage on a Windows Server 2008 R2 VM with SQL Server 2008 SP1 (MaxPatchCacheSize set to 0)"
class="screenshot" height="493" width="600"
title="Figure 2: Disk usage on a Windows Server 2008 R2 VM with SQL Server 2008 SP1 (MaxPatchCacheSize set to 0)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/After-restricting-MaxPatchCacheSize-857x704.png)

Observe that the $PatchCache$ folder isn't even visible in the disk usage view,
and the free space on the VHD is substantially larger. You can imagine how the
delta between the two configurations would increase substantially as more
products -- and thus more patches -- are installed.

As noted on MSDN, the registry setting "does not affect files that have already
been saved." In other words, if you set MaxPatchCacheSize to 0 after installing
a bunch of products and patches, then you're pretty much stuck with the "wasted"
disk space -- unless you use something like MSIZAP to clean up the Installer
files.

> **Important**
>
> Setting MaxPatchCacheSize to 0 doesn't come without penalty. There are rare
> occasions where you have to do a little more work when installing new products
> or features.\
> \
> For example, while testing my upgrade from TFS 2008 to TFS 2010 on a new set
> of VMs, I first installed SharePoint Server 2010 on one of the VMs. Then I
> subsequently needed to install Reporting Services on the same VM (for the TFS
> reports). Since I had already installed the SQL Server 2008 client piece (and
> patched it to SP1), when I went to add Reporting Services, SQL Server Setup
> complained about not being able to find the version of sqlncli.msi it needed
> (because it was installed from a temporary folder via Windows Update but since
> my MaxPatchCacheSize was set to 0, it wasn't copied to the
> \Windows\Installer\$PatchCache$ folder).\
> \
> Consequently, I simply had to extract the service pack (using the /EXTRACT
> command-line parameter) and search around a little for the expected file. Once
> I located the version it needed, SQL Server Setup continued on its way and
> completed successfully.

Note that I don't set MaxPatchCacheSize to 0 on physical machines (although I
thought about it recently when setting up my laptop with a new 80GB SSD drive).
Ultimately, it's a judgment call that you need to make based on how much disk
space you are willing to trade for the convenience of simpler software and patch
installations.
