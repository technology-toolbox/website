---
title: 'Active Directory Domain Structure in the "Jameson Datacenter"'
date: 2009-10-02T07:50:00-06:00
excerpt:
  'In a previous post, I provided some details on the "Jameson Datacenter" ,
  which is really just my home lab that I use for learning new technologies and
  improving my skills, as well as actually completing my day-to-day tasks on
  various customer projects...'
aliases:
  [
    "/blog/jjameson/archive/2009/10/01/active-directory-domain-structure-in-the-jameson-datacenter.aspx",
    "/blog/jjameson/archive/2009/10/02/active-directory-domain-structure-in-the-jameson-datacenter.aspx",
  ]
categories: ["My System", "Infrastructure"]
tags: ["My System", "Infrastructure"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/10/02/active-directory-domain-structure-in-the-jameson-datacenter.aspx"
---

In a previous post, I provided some details on the
["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter), which
is really just my home lab that I use for learning new technologies and
improving my skills, as well as actually completing my day-to-day tasks on
various customer projects (whenever I am actually able to work from home instead
of using my laptop at a customer site).

In this post, I want to share details about how I've configured my Active
Directory domain, namely **corp.technologytoolbox.com**.

{{< div-block "note" >}}

> **Note**
>
> Technology Toolbox is just a name I came up with about ten years ago before
> joining Microsoft. At the time, I was considering starting my own company and
> therefore went looking for an available domain name. (I would have preferred
> techtoolbox.com, but that was already taken.)
> [Don't bother trying to browse to [https://www.technologytoolbox.com](https://www.technologytoolbox.com/)
> because, honestly, I haven't yet invested the time in actually creating the
> Web site -- even after almost ten years.]

{{< /div-block >}}

As you can see in the following figure, I've created a number of organizational
units (OUs) which you would typically find in many enterprise organizations.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Jameson-Datacenter-AD-domain-structure-302x600.png"
alt="Jameson Datacenter - Active Directory domain structure" class="screenshot"
height="600" width="302"
title="Figure 1: Jameson Datacenter - Active Directory domain structure" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Jameson-Datacenter-AD-domain-structure-305x605.png)

Note that I didn't really come up with this structure myself (except for the
**Admin Accounts** OU, which I believe was an
["original idea"](http://en.wikipedia.org/wiki/A_Beautiful_Mind_%28film%29) of
mine -- but honestly it's been so long that I can hardly remember). Most of this
is based on the prescriptive guidance from TechNet. You can find the latest
version of this guidance in the
[AD DS (Active Directory Domain Services) Design Guide](http://technet.microsoft.com/en-us/library/cc754678%28WS.10%29.aspx).
[I tried to find the original reference material that I used, but I gave up
looking for it on TechNet this morning after about 5 minutes.]

As you can see, I created OUs to model organizations such as the **Development**
group, the **IT** group, and the **Sales** organization.

The following table summarizes the various OUs in the domain:

<div class="d-sm-none">
  <a href='{{< relref "resources/table-1-popout" >}}' target="_blank">Table 1 - Organizational Units in the corp.technologytoolbox.com Domain</a>
  <i class="bi bi-arrow-up-right-square"></i>
  <p>(Insufficient width to show table content here.)</p>
</div>
<div class="d-none d-sm-block">
  {{< include-html "resources/table-1.html" >}}
</div>

{{< div-block "note" >}}

> **Note**
>
> The **Sales** OU contains similar OUs as **Development**. These are simply not
> shown in the previous table.

{{< /div-block >}}

Note that this domain structure, I'm able to use the Group Policy feature of
Active Directory to "effortlessly" configure new servers.

For example, this morning I created a new development VM (FOOBAR3) from a
[SysPrep'ed VHD](/blog/jjameson/2009/08/13/using-sysprep-ed-vhds-for-new-hyper-v-virtual-machines)
file. After joining the new VM to the domain, I then moved FOOBAR3 to the
**Development/Resources/Servers** OU. After the VM rebooted (which is required
when you join a domain), it was automatically configured with numerous settings,
including Windows Update (i.e. to update from my WSUS server -- COLOSSUS) and
also to grant the **TECHTOOLBOX\Development Admins** group administrative access
(of which **TECHTOOLBOX\jjameson** is a member). I was then able to quickly log
in with my regular account and start installing SQL Server 2008 (which finished
before I actually completed this post). Now it's time to install Visual Studio
2008...

I'll cover my group policy configuration in a separate post.
