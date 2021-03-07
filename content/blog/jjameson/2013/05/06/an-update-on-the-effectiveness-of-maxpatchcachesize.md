---
title: "An Update on the Effectiveness of MaxPatchCacheSize"
date: 2013-05-06T23:22:03-06:00
lastmod: 2013-05-06T23:25:31-06:00
excerpt: "Wondering where all the precious free space in your SSD went? Well, here are a few possibilities."
aliases: ["/blog/jjameson/archive/2013/05/06/an-update-on-the-effectiveness-of-maxpatchcachesize.aspx"]
draft: true
categories: ["Infrastructure", "My System"]
tags: ["Infrastructure", "My System", "Virtualization", "Visual Studio"]
---

In 	[my previous post](/blog/jjameson/2013/05/06/powershell-scripts-for-managing-maxpatchcachesize), I shared a couple of PowerShell scripts that I wrote for  	quickly setting and verifying the MaxPatchCacheSize registry setting. I also  	stated that this registry setting has, unfortunately, become less effective  	over the years since 	[my original post on this subject](/blog/jjameson/2010/04/30/save-significant-disk-space-by-setting-maxpatchcachesize-to-0).

For example, here's what the disk usage looks like on one of my development  	VMs:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-Usage-FOOBAR7-Baseline-600x460.png"
alt="Disk usage"
class="screenshot"
height="460"
width="600"
title="Figure 1:" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-Usage-FOOBAR7-Baseline-991x760.png)

Note that the \Windows\Installer folder is consuming almost 5.5 GB of space  	on the VHD -- despite the fact that I set MaxPatchCacheSize to 0 immediately  	after creating the VM.

5.5 gigabytes isn't all that much when your hard drive is several hundred  	gigabytes -- or perhaps even a terabyte or two -- in size.

However, the same number is a different matter altogether on VHDs that you  	are trying to keep as small as possible. For example, maybe like me, you have  	a number of VMs running on SSDs. In that case, multiplying the "wasted space"  	in the Windows Installer folder by the number of VMs is, well, *aggravating*.

Also note from the screenshot above there's a considerable amount of disk  	space consumed by the following folders:

- C:\ProgramData\Package Cache
- C:\Users\All Users\Package Cache
- C:\Program Files\Microsoft SQL Server\110\Setup Bootstrap\Update
  Cache

In other words, not only is a significant amount of space being  	wasted in the Windows Installer folder (which is supposed to be controllable  	using the MaxPatchCacheSize setting), now we also have several other locations  	that chew up precious cells on SSDs.

In this particular case, the bloat in the Package Cache folders is due to  	installing Visual Studio 2012.

Unfortunately, it seems like this is just the way it's going to be from now  	on -- at least according to Heath Stewart from Microsoft:

{{< reference title="How Visual Studio 2012 Avoids Prompts for Source" linkHref="http://blogs.msdn.com/b/heaths/archive/2012/07/26/how-visual-studio-2012-avoids-prompts-for-source.aspx" >}}

Heath, if you're reading this, then sorry, but I think Microsoft's decision  	to force this bloat on us is, um, stupid. Okay, maybe *shortsighted*  	is a softer way to put it. I don't deny the numbers you reference in your blog  	post; I just wish you provided a better response to the SSD scenario (especially  	in light of the fact that many folks rely heavily on virtualization these days).

All I'm asking for is to make the installation caches configurable. In other  	words, something like the MaxPatchCacheSize registry setting -- except something  	that actually works as intended ;-)

> **Note**
>
> The development VM that I captured the above screenshot from has the
> following software installed:
>
> - Windows 7 (64-bit)
>
> - Microsoft Visual Studio 2008
>
> - Microsoft Visual Studio 2008 Team Explorer
>
> - Microsoft Visual Studio 2008 SP1
>
> - Microsoft Visual Studio 2010
>
> - Microsoft Visual Studio 2010 SP1
>
> - Microsoft Team Foundation Server Power Tools December 2011
>
> - Microsoft Visual Studio 2012
>
> - Microsoft Visual Studio 2012 Update 2
>
> - Microsoft SQL Server 2012 with SP1
>
> - WiX Toolset v3.7
>   
>       	I don't typically install more than one version of Visual Studio in 
>       	a VM. However, I built this "uber Visual Studio VM" in order to debug 
>       	some issues with the
>       	[TFS Integration](http://visualstudiogallery.msdn.microsoft.com/eb77e739-c98c-4e36-9ead-fa115b27fefe) tool (which I use to synchronize my local TFS team 
>       	projects with my clients' TFS instances).

