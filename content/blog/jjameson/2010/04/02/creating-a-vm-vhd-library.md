---
title: "Creating a VM/VHD Library"
date: 2010-04-02T07:37:00-06:00
excerpt: "Last week a colleague was asking me how I manage my various VMs. More specifically, he wanted to know how I created SysPrep'ed images in order to quickly \"spin up\" new VMs for development, testing, or demo purposes. 
 Note that I like to keep my environments..."
aliases: ["/blog/jjameson/archive/2010/04/01/creating-a-vm-vhd-library.aspx", "/blog/jjameson/archive/2010/04/02/creating-a-vm-vhd-library.aspx"]
draft: true
categories: ["My System", "Infrastructure"]
tags: ["My System", "Virtualization"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/02/creating-a-vm-vhd-library.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/02/creating-a-vm-vhd-library.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Last week a colleague was asking me how I manage my various VMs. More
specifically, he wanted to know how I created SysPrep'ed images in order to
quickly "spin up" new VMs for development, testing, or demo purposes.

Note that I like to keep my environments as "clean" as possible. Consequently, I
don't want my SysPrep'ed image of, say, Windows Server 2008 Service Pack 2 to
have all of the patches that were released through Windows Update prior to the
release of SP2. However, I also don't want to spend a lot of time waiting for
Windows Update to apply dozens of patches each time I create a new VM.
Installing a few patches after firing up a new VM isn't a big deal, provided the
Windows Update process doesn't take more than 5 minutes or so.

Let's suppose you were just starting your VM library today. Here's the high
level overview of the process that I recommend you follow. Note that it doesn't
really matter whether you are using Hyper-V, Virtual PC, or some other
virtualization platform -- the general approach is the same.

To add a base Windows Server 2008 Standard Edition x86 VM to your library:

1. Create a new virtual machine named **ws2008std-x86** with a dynamically expanding VHD. [Note that I used to set the maximum size of the primary VHD file to 16 GB (back in the days of Windows Server 2003) but now I typically set the maximum size to 20 GB. Remember that you can always expand it later as necessary, and you don't even need tools like Partition Magic anymore in order to increase the size of the partition because this "just works" in Windows Server 2008 directly from the Disk Management MMC snap-in.]
2. Insert your DVD for Windows Server 2008 Standard Edition (x86) and install the operating system. I do not recommend adding any server roles or features at this point, because, honestly, I've just found there to be lots of issues when you SysPrep after installing more than just the base OS. Many times you can workaround these issues, but most of the time my preference is to avoid the problems altogether.
3. After the OS is installed, run the SysPrep utility and specify the option to **Enter System Out-of-Box Experience (OOBE)** and **Shutdown** when SysPrep is done. [Note that for Windows Server 2003 you had to copy the SysPrep folder from a CAB file on the installation media (or download it from microsoft.com), but thankfully this extra step is no longer necessary in Windows Server 2008.]
4. After the VM shuts down, immediately make a copy of the VHD file to a new file named **ws2008std-x86\_RTM.vhd**.
5. Boot the VM, go through the Mini-Setup process and once again name the server **ws2008std-x86**.
6. Next, install SP2 for Windows Server 2008. Then run the Windows Component Clean Tool (COMPCLN.exe) to [reclaim some disk space after installing SP2](/blog/jjameson/2009/06/02/reclaiming-disk-space-after-installing-service-pack-2) and reboot the VM.
7. Run the SysPrep utility again and specify the same options as before.
8. After the VM shuts down, immediately make a copy of the VHD file to a new file named **ws2008std-x86\_SP2.vhd**.
9. Boot the VM again and go through the Mini-Setup process (once again naming the server **ws2008std-x86**).
10. Run Windows Update, install all of the latest patches, and then reboot.
11. Run the SysPrep utility one last time (specifying the same options as before).

At this point, you have clean images for the following configurations:

- Windows Server 2008 x86 RTM (note that for Windows Server 2008, SP1 was actually included in the initial release)
- Windows Server 2008 x86 SP2
- Windows Server 2008 x86 SP2 with all of the latest patches

Now you can simply create a new VM (for example, named **foobar**) and select
the option to create a new VHD for the VM. However, before booting the new VM
for the first time, copy **ws2008std-x86.vhd** to **foobar.vhd**, thus starting
from a SysPrep'ed image of Windows Server 2008 x86 SP2 with all of the latest
patches.

Note that there might be a scenario where you are trying to identify an issue
that only occurs after installing SP2 or one of the recent patches from Windows
Update. In this case, you would copy **ws2008std-x86\_RTM.vhd** or
**ws2008std-x86\_SP2.vhd**, respectively, when overwriting **foobar.vhd**.

In other words, the "snapshots" of the VHD file allow you to quickly go back to
different baseline versions of the OS. Don't confuse the word "snapshots" that
I've used here with the term as it applies to Hyper-V. [I definitely don't
recommend using Hyper-V snapshots in order to support this particular scenario
(they are great for other scenarios -- just not this one).]

Suppose that some time goes by and you find that it takes longer than you would
like to install the latest patches from Windows Update whenever you create a new
VM. To resolve this issue, simply boot the **ws2008std-x86** VM, run Windows
Update, reboot, and then run SysPrep again. The next time you create a new VM
starting from a copy of **ws2008std-x86.vhd**, you won't have to wait long at
all for Windows Update to finish.

Let's fast forward to some future point in time when Windows Server 2008 SP3 is
released. Here is what I will do in the
["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) (a.k.a.
my home lab) at that point:

1. Overwrite the **ws2008std-x86.vhd** file with **ws2008std-x86\_RTM.vhd** (thereby "purging" the baseline VM image of all of the patches and updates).
2. Boot the VM, go through the Mini-Setup process and name the server **ws2008std-x86**.
3. Install SP3 for Windows Server 2008 and, presumably, run the Windows Component Clean Tool (COMPCLN.exe) to reclaim some disk space, and reboot the VM.
4. Run the SysPrep utility and specify the option to **Enter System Out-of-Box Experience (OOBE)** and **Shutdown** when SysPrep is done.
5. After the VM shuts down, immediately make a copy of the VHD file to a new file named **ws2008std-x86\_SP3.vhd**.

I should point out that your **ws2008std-x86** image isn't going to do you much
good if you need to install SharePoint Server 2010 (because it is 64-bit only).
Therefore, like me, you should consider creating additional VM images in your
library, such as:

- ws2008std-x64
- ws2008std-r2

In the Jameson Datacenter, I currently have an image for Win2k3EE (Windows
Server 2003 Enterprise Edition), but I don't use it very often anymore. Note
that most of the time, all of these "base" VMs are shutdown. I only start them
when a Service Pack is released or when Windows Update exceeds my personal
threshold while creating a new VM.

There are a couple of details that I should point out with regards to using this
process with Hyper-V. This is because Hyper-V isn't quite as "friendly" as
Virtual PC when it comes to copying a VM (or VHD).

The first issue is related to permissions on the VHD file, which I've already
covered in a
[previous post](/blog/jjameson/2009/08/13/using-sysprep-ed-vhds-for-new-hyper-v-virtual-machines).
The second is that you will likely end up with extra network connections (e.g.
**Local Area Connection 2**). As long as you are using DHCP, these extraneous
network connections shouldn't cause any issues. However, if you want to remove
them simply follow the steps detailed in
[KB 269155](http://support.microsoft.com/kb/269155) (**Method 1**).

