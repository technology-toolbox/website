---
title: "Some Gotchas with Remote Administration of Hyper-V"
date: 2008-08-28T08:41:00-06:00
excerpt:
  "As I mentioned in my previous post , last month I built out a new virtual
  environment using Hyper-V on Server Core. Since you can't run MMC -- and
  therefore Hyper-V Manager -- on Server Core, you need to use remote
  administration to manage the VMs. 
  ..."
aliases: ["/blog/jjameson/archive/2008/08/27/some-gotchas-with-remote-administration-of-hyper-v.aspx", "/blog/jjameson/archive/2008/08/28/some-gotchas-with-remote-administration-of-hyper-v.aspx"]
draft: true
categories: ["Infrastructure"]
tags: ["Windows Server", "Infrastructure", "Virtualization"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2008/08/28/some-gotchas-with-remote-administration-of-hyper-v.aspx](http://blogs.msdn.com/b/jjameson/archive/2008/08/28/some-gotchas-with-remote-administration-of-hyper-v.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

As I mentioned in my
[previous post](/blog/jjameson/2008/07/07/copy-paste-gotchas-with-server-core),
last month I built out a new virtual environment using Hyper-V on Server Core.
Since you can't run MMC -- and therefore Hyper-V Manager -- on Server Core, you
need to use remote administration to manage the VMs.

John Howard's
[blog series on Hyper-V Remote Management](http://blogs.technet.com/jhoward/archive/2008/03/28/part-1-hyper-v-remote-management-you-do-not-have-the-requested-permission-to-complete-this-task-contact-the-administrator-of-the-authorization-policy-for-the-computer-computername.aspx)
is by far the definitive source for getting Hyper-V up and running on Server
Core. It provides an excellent step-by-step guide for enabling remote
administration, opening various firewall ports, configuring DCOM permissions (if
you don't want to use admin accounts), etc. If you haven't yet at least scanned
John's posts, I highly recommend doing so before embarking on the Hyper-V on
Server Core path.

There is one snag, however, that I want to point out with regards to John's
scenarios.

The instructions in John's blog posts work when:

- both the Hyper-V server and the client are in WORKGROUP mode, or
- when the client and server are members of the same domain or trusted domains,
  or
- when the Hyper-V server is in WORKGROUP but the client is in a domain

[Note that I personally verified the second and third scenarios above while
initially building out my Hyper-V server; I am trusting that the first scenario
works based on John's posts.]

However, if the client is a member of DOMAIN1 and the Hyper-V server is a member
of DOMAIN2 -- and there is no trust relationship between DOMAIN1 and DOMAIN2 --
then Hyper-V Manager pukes with a message about not being able to connect to the
RPC server. Also note that in this scenario Disk Management pukes as well with
the infamous error message:

{{< blockquote "font-italic text-danger" >}}

RPC server is unavailable

{{< /blockquote >}}

To picture this scenario, imagine you have a Hyper-V server joined to your
internal domain, but now I come along and try to use Hyper-V Manager from my
laptop which is joined to the internal Microsoft domain. It simply doesn't work
-- and neither does Disk Management.

At this point, you might be thinking something like "Jeremy, it sounds like a
firewall issue or you haven't enabled Remote Volume Management." However,
immediately after receiving the "RPC server is unavailable" message on my
laptop, I was able to connect the Disk Management console to the Hyper-V server
just fine from a Windows Server 2003 member server in the same domain.

In my mind, that indicated the firewall and remote administration were
configured correctly. After a little research, it appeared that I was hitting a
known bug with WMI when the client and server are in different, untrusted
domains.

To workaround this issue, I did two things.

First, I created a new Vista VM and joined it to my customer's internal domain.
After installing the
[Hyper-V Remote Management Update for Windows Vista (KB952627)](http://www.microsoft.com/downloads/details.aspx?familyid=BF909242-2125-4D06-A968-C8A3D75FF2AA&displaylang=en),
I was able to start, stop, and create VMs on the Hyper-V server. Excellent.

Second, since I didn't want to have to always fire up my Vista VM just to view
or change the Hyper-V settings on the server, I installed the
[Hyper-V Update for Windows Server 2008 x64 Edition (KB950050)](http://www.microsoft.com/downloads/details.aspx?FamilyID=f3ab3d4b-63c8-4424-a738-baded34d24ed&DisplayLang=en)
on one of the VMs running on the Hyper-V server. This obviously doesn't
completely replace the need for a remote administration client due to the
"Catch-22" scenario -- meaning, if the VM isn't running, you can't use Hyper-V
Manager from the VM to start the VM ;-)

Someday soon, I'm hoping we will release a few command line tools for Hyper-V
that allow you to perform some basic operations such as starting or stopping
VMs. This would be great on Server Core -- and no, I don't want to install
PowerShell in order to do this ;-) [In keeping with the spirit of Server Core, I
want to install as little as possible on the host.]

Lastly, I want to point out one of the stumbling blocks that I encountered along
the way. Before I actually created the Vista VM that I mentioned earlier for
remote administration, I initially created a Windows Server 2008 x86 VM and
installed the 32-bit version of
[KB950050](http://www.microsoft.com/downloads/details.aspx?FamilyId=6F69D661-5B91-4E5E-A6C0-210E629E1C42&displaylang=en)
in order to use Hyper-V Manager to remotely administer the Hyper-V server.

According to the corresponding
[KB article](http://support.microsoft.com/kb/950050):

> **Update for Windows Server 2008 (KB950050)**\
> This 32-bit update package includes the release version of the following:
>
> - The Hyper-V Manager console
> - The Virtual Machine Connection tool for x86-based editions of Windows Server
>   2008

Based on my experience installing the
[Hyper-V Remote Management Update for Windows Vista (KB952627)](http://www.microsoft.com/downloads/details.aspx?familyid=BF909242-2125-4D06-A968-C8A3D75FF2AA&displaylang=en)
and the note above from the KB article, after installing KB950050 on Windows
Server 2008 I expected to be able to start MMC and add the Hyper-V Manager
snap-in. However, it doesn't quite work that way.

Fortunately, I received a quick response to my inquiry from Alex Kibkalo, a
fellow Architect with Microsoft Consulting Services in Russia.

To enable Hyper-V Manager after installing KB950050, you need to enable the
corresponding feature:

1. Open **Server Manager**. (If Server Manager is not running, click **Start**,
   point to **Administrative Tools**, click **Server Manager**, and then, if
   prompted for permission to continue, click **Continue**.)
2. In **Server Manager**, under **Features Summary**, click **Add Features**.
3. In the **Add Features Wizard**, on the **Select Features** page, expand
   **Remote Server Administration Tools**, and then expand **Remote
   Administration Tools**
4. Click **Hyper-V Tools**, and then proceed through the rest of the wizard.

For more information on deploying Hyper-V, refer to the
[Hyper-V Planning and Deployment Guide](http://www.microsoft.com/downloads/details.aspx?familyid=5DA4058E-72CC-4B8D-BBB1-5E16A136EF42&displaylang=en).
