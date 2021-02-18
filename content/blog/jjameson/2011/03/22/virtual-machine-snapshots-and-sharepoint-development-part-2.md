---
title: "Virtual Machine Snapshots and SharePoint Development, Part 2"
date: 2011-03-22T23:51:00-07:00
excerpt: "In part 1 of this post, I introduced the way I use VM snapshots to allow me to quickly rollback my SharePoint development VMs to key points in time. For example, I can quickly revert to a \"baseline SharePoint Server 2010 configuration\" in which no Web..."
aliases: ["/blog/jjameson/archive/2011/03/22/virtual-machine-snapshots-and-sharepoint-development-part-2.aspx"]
draft: true
categories: ["My System", "SharePoint", "Infrastructure"]
tags: ["My System", "MOSS 2007", "
                    Windows Server", "Virtualization", "SharePoint 2010"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/03/23/virtual-machine-snapshots-and-sharepoint-development-part-2.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/03/23/virtual-machine-snapshots-and-sharepoint-development-part-2.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

In [part 1](/blog/jjameson/2011/03/21/virtual-machine-snapshots-and-sharepoint-development-part-1) of this post, I introduced the way I use VM snapshots to allow me         to quickly rollback my SharePoint development VMs to key points in time. For example,         I can quickly revert to a "baseline SharePoint Server 2010 configuration" in which         no Web applications have been created (besides Central Administration) and the only         service application that has been configured is [Usage and Health Data Collection](http://technet.microsoft.com/en-us/library/ee663480.aspx).

At the end of part 1, I indicated that there was still a lot I wanted to cover on         this topic.

However, before discussing my recommendations for effectively using snapshots, let's         first review what makes it feasible to use Hyper-V snapshots for SharePoint development         environments to begin with:

- Hyper-V host running Windows Server 2008 R2 (preferably Server Core edition)
- Domain controller running on separate VM (or better yet, multiple domain controllers
  on multiple VMs)
- SQL Server for SharePoint farm running on same VM (and no other SharePoint servers
  in the farm)

### Hyper-V Host Running Windows Server 2008 R2

You certainly don't have to run Windows Server 2008 R2 on the Hyper-V host in order         to create snapshots. In other words, you could just choose to run Windows Server         2008 on the host instead. However, I strongly recommend running the newer release         for a couple of reasons.

Before WS2008 R2, Hyper-V snapshot data files were not created side-by-side with         their corresponding VHD files, but rather were located by default in the same folder         as the virtual machine. Consequently if you utilize multiple VHDs for a given VM         (for example, [C:, D:, and L: virtual hard drives](/blog/jjameson/2011/03/18/cdl-for-sharepoint-a-k-a-quot-you-can-never-have-too-many-spindles-quot)) and you spread the VHDs across multiple         physical drives -- which is something I definitely recommend -- then if you create         a snapshot in WS2008, the .avhd files aren't spread across the physical spindles         like the original VHD files. This can also cause you to unexpectedly run out of         disk space since the .avhd growth doesn't always take place on the same physical         drives as the corresponding VHD files. Fortunately, in WS2008 R2, the .avhd files         are created side-by-side with their VHD counterparts.

The other reason you'll want to be sure to run WS2008 R2 on the Hyper-V host --         and this is really one of the most important improvements to understand about the         Hyper-V R2 release -- is the fact that disk performance is substantially improved         in WS 2008 R2. If you haven't at least skimmed the following white paper, I recommend         taking a look at it:

{{< reference title="Virtual Hard Disk Performance - Windows Server 2008 / Windows Server 2008 R2 / Windows 7" linkHref="http://download.microsoft.com/download/0/7/7/0778C0BB-5281-4390-92CD-EC138A18F2F9/WS08_R2_VHD_Performance_WhitePaper.docx" >}}

In my experience, [VM performance is typically throttled by I/O](/blog/jjameson/2007/06/23/performance-of-virtual-machines) -- rather than CPU, memory,         or network resources. Sure, there are exceptions to this, but generally speaking,         slow VMs -- that otherwise appear healthy in terms of CPU and memory usage -- are         usually caused by disk contention on the underlying physical disks.

After you create snapshot of a VM, performance takes an additional hit due to the         need to read data from (potentially multiple) .avhd files and, assuming the data         hasn't been updated since the snapshot was taken, the original .vhd file itself.

Okay, enough ranting about I/O for this post.

If you're not sure why I recommend running Server Core edition on the Hyper-V host,         it's really all about memory...the less RAM consumed by the host, the more memory         available for running virtual machines. Oh, and the [decreased attack surface](/blog/jjameson/2009/06/04/why-choose-server-core-installation-of-windows-server-2008) is also a nice bonus :-)

### Domain Controller Running on Separate VM

The second item I noted before regarding making snapshots feasible is to ensure         that you run a separate domain controller (or preferably multiple domain controllers).         Sure, you *can* choose to run Active Directory locally on the same server         as SharePoint and still use snapshots -- provided the VM is completely isolated         (meaning no other machines are joined to the domain). However, as soon as you forego         the complete isolation of your domain controller, you also forego snapshots. In         other words, this is simply keeping within the supportability constraints of snapshots         and Active Directory domain controllers.

The short explanation is simply "don't do it." If you want a longer explanation,         go search TechNet.

### SQL Server for SharePoint Farm Running on Same VM

Similar to the previous section, the fact that SQL Server and SharePoint are running         on the same VM (and there are no other SharePoint servers in the farm) means snapshots         can be used without fear of wreaking havoc on your SharePoint environment.

Imagine if this weren't the case. For example, your SharePoint environment consists         of two VMs (one for SharePoint Server 2010 and the other for SQL Server 2008) or         perhaps the environment consists of three or more VMs with multiple SharePoint servers,         etc.

If you try to use snapshots in those scenarios, you are simply asking for trouble         -- and I'm fairly certain you are officially "unsupported" if you do. The reason         is due to the fact that unless you can simultaneously snapshot all of the servers         in the farm -- and subsequently ensure that you *always* revert all servers         in the farm to the same point in time -- then you'll likely end up "corrupting"         your farm due to the fact that timer jobs are firing off constantly in SharePoint         (I'm exaggerating this a little, but not much) and you need to be sure that what         the various SharePoint databases say is the "truth" indeed matches the state on         the various servers in the farm. This is easy to achieve when SharePoint and SQL         reside on the same VM -- and there are no other SharePoint servers in the farm.         Otherwise, you should forget about using snapshots (at least in my opinion).

### Recommendations for Effectively Using Snapshots

With the prerequisites out of the way, let's move on to my recommendations for effectively         using snapshots.

As a quick review, the following screenshot shows the snapshots for my primary SharePoint         Server 2010 development VM (FOOBAR5):

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/9/r_FOOBAR5%20Snapshots.png"
alt="FOOBAR5 snapshots"
height="383"
width="600"
title="Figure 1: FOOBAR5 snapshots" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/9/o_FOOBAR5%20Snapshots.png)

For the sake of explaining more about these snapshots, I'll relabel them again in         this post as follows:

- Baseline SharePoint Server 2010 configuration - **Snapshot 1**
  - Baseline Fabrikam Demo Site (SharePointClaimsAuthentication) - **Snapshot
    2**
  - Baseline {Client Name} Cloud Portal (Sprint-11) - **Snapshot 3**

#### Think "Shallow Snapshot Trees" for Better Performance

Notice that "Snapshot 3" is based on "Snapshot 1" rather than "Snapshot 2". In addition         to isolating the configuration of my "Fabrikam Demo Site" from my client environment         (i.e. the "Cloud Portal"), I reverted back to "Snapshot 1" before proceeding to         follow the installation guide to deploy the "Cloud Portal" solution.

While the isolation is somewhat important, the primary reason I structured the snapshots         this way is to minimize the number of differencing disks that need to be read when         performing my primary day-to-day work.

Consider the alternative snapshot structure:

- Baseline SharePoint Server 2010 configuration - **Snapshot 1**
  - Baseline Fabrikam Demo Site (SharePointClaimsAuthentication) - **Snapshot
    2**
    - Baseline {Client Name} Cloud Portal (Sprint-11) - **Snapshot 3**

If I had used this snapshot structure instead, then my VM would have to read an         additional three .avhd files when running the "Cloud Portal" solution (since my         VM is configured with C:, D:, and L: virtual hard drives).

In other words, as soon as I created "Snapshot 1", my original VHD files became         read only and a set of three .avhd files were created (one each for C:, D:, and         L:). At the point where I created "Snapshot 2", those original .avhd files became         read only and an additional three were created. Thus when running the VM from "Snapshot         2", two sets of differencing .avhd files must be read (in addition to the original         VHD files).

By creating "Snapshot 3" after first reverting to "Snapshot 1", I alleviate the         need to read the .avhd files for "Snapshot 2" while running my primary development         configuration.

#### Revert to Snapshots Frequently

The whole point of creating snapshots is to be able to rollback to an earlier point         with minimal time and effort. Consequently, as I first mentioned in a [previous post](/blog/jjameson/2011/02/26/deployment-scripts-for-sharepoint-server-2010), I like to treat my SharePoint development VM as "volatile"         -- or "disposable" (if you prefer that term instead). This obviously doesn't mean         that I'm willing to "throw away" the entire VM (considering the effort spent setting         it up in the first place). Rather it means that I periodically discard the "Now"         configuration (i.e. the changes in the VM after starting from a previous snapshot).

Of course, this emphasizes the importance of being able to deploy your SharePoint         solutions (and configure changes on the site) with minimal effort, which I typically         characterize as running some deployment scripts in rapid succession.

It's also worth repeating the following warning from my previous post:

> **Warning**
>
> If, like me, you decide to use Hyper-V snapshots in your SharePoint development environment, then make darn sure you've checked in any pending changes to TFS (or at least shelved your changes) before you apply a snapshot. [Note that when applying an earlier snapshot, I don't typically take a new snapshot before reverting to the earlier point in time and hence any work I've done that hasn't be "exported" somewhere (e.g. checked into TFS) is subsequently lost when I apply a snapshot.]

#### Create a New Snapshot After Each Release

In my mind there are two scenarios where snapshots provide the most value.

The first is when something "really bad" happens in your SharePoint development         environment and you find yourself spending nearly an hour or more trying to resolve         the issue. It's amazing how quickly an hour goes by when troubleshooting an issue         in SharePoint. Fortunately, this doesn't happen very often, but on the rare occasion         that it does, it is wonderful to be able to just "nuke" the current state of the         VM and rollback to a "last known good configuration" (which in this case I am referring         to a snapshot).

The other scenario where snapshots provide tremendous value is when deploying frequent         releases of your solution. In other words, when you are using short iterations or         sprints on your project. By applying a snapshot corresponding to the state of your         solution at the end of your previous iteration, you can quickly test your upgrade         process to deploy the newer version from your current iteration.

To understand this process, examine the snapshots shown below for one of my Microsoft         Office SharePoint Server 2007 development VMs (FOOBAR2):

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/9/r_FOOBAR2%20Snapshots.png"
alt="FOOBAR2 snapshots"
height="372"
width="600"
title="Figure 2: FOOBAR2 snapshots" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/9/o_FOOBAR2%20Snapshots.png)

On my current project, we are working on Sprint-12 which includes some enhancements         to the "Client Portal". The previous release of the solution was Sprint-10. [If         you are wondering what happened to Sprint-11, then take another look at Figure 1         above -- which shows how we devoted that iteration to setting up a new SharePoint         Server 2010 collaboration platform.]

As we move through Sprint-12, I periodically revert back to the "Sprint 10" snapshot         and then deploy the latest Sprint-12 build to that environment. This ensures the         deployment "just works" when it comes time to deploy to the Test environment (and         eventually to Production).

Once Sprint-12 is complete and the build is released to Production, I'll revert         one more time to the "Sprint-10" snapshot, follow the installation guide to deploy         the final build for Sprint-12, take a new snapshot (naming it something like "Sprint-12"),         and subsequently delete the old "Sprint-10" snapshot.

It might be tempting to create a snapshot structure similar to the following:

- Sprint-1
  - Sprint-2
    - Sprint-3
      - ...

However, that would result in a large number of .avhd files and therefore an unbearable         amount of I/O "thrashing." Instead, I choose to keep a snapshot corresponding to         the most recent release to Production. After all, once Sprint-12 has been released         to PROD, do you think anyone will really care about the Sprint-10 release anymore?

#### Consider Rebuilding Snapshots When Installing Service Packs

With the recent release of Windows Server 2008 R2 Service Pack 1, I was concerned         about the performance of my development VMs if all of the changes in SP1 were applied         to the .avhd files instead of the original VHDs. Consequently, I reverted my various         development VMs back to the first snapshot (e.g. "Baseline MOSS 2007 configuration")         and then deleted the snapshot tree (thereby deleting all of the differencing disks).         Thus when I installed the operating system service pack, all of the changes were         written to the main VHD for each VM.

For each VM, I then created a new snapshot using the original name (e.g. "Baseline         MOSS 2007 configuration") and then proceeded to deploy the latest release of the         solution (e.g. "Sprint-10") and take another snapshot.

This obviously isn't ideal, but personally I'd rather do this in order to keep the         .avhd files as small as possible.

### Alternatives to Using Snapshots

Note that prior to using Hyper-V snapshots, I used to create SharePoint backups         corresponding to each release. In other words, instead of taking a snapshot after         deploying the final build of "Sprint-10", I take a SharePoint backup (of the entire         farm).

This works reasonably well -- and, in fact, that process is still used in our Test         environment since it is comprised of multiple VMs (one for SQL Server and two SharePoint         servers). However, it does require more work to revert to a previous point in time.         For example, an additional database restore must be performed on our custom database         -- since that obviously isn't included in the SharePoint backup. We also have to         manually "DR.D" (deactivate, retract, and delete) the custom SharePoint features         and solutions and subsequently "ADA" (add, deploy, and activate) the corresponding         versions that match the SharePoint backup.

This is certainly doable...it just takes more work than clicking the **Apply...** menu item in Hyper-V Manager to revert a development VM to a snapshot.

Uh oh...look at the time! I have to get my daughter off to school. Fortunately,         I believe I've covered everything I intended to in this post. I hope you find Hyper-V         snapshots as valuable as I do when working with SharePoint.

