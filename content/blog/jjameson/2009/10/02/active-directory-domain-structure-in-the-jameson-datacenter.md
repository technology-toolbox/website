---
title: "Active Directory Domain Structure in the \"Jameson Datacenter\""
date: 2009-10-02T07:50:00-06:00
excerpt:
  "In a previous post, I provided some details on the \"Jameson Datacenter\" ,
  which is really just my home lab that I use for learning new technologies and
  improving my skills, as well as actually completing my day-to-day tasks on
  various customer projects..."
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

{{< table class="small table-striped"
caption="Organizational Units in the corp.technologytoolbox.com Domain" >}}

| Organizational Unit | Description |
| --- | --- |
| Development | Represents the "Development" group within Technology Toolbox (which really just means me, since my wife doesn't do any development and my 3-1/2 year old daughter isn't quite ready for .NET yet) |
| Development/Groups | Contains the groups within the Development organization (such as **TECHTOOLBOX\All Developers** and **TECHTOOLBOX\Development Admins**). Delegated administration is used to allow developers to create and manage their own groups. |
| Development/Resources | Container for various resource OUs |
| Development/Resources/Servers | Contains servers for the Development environment (e.g. **DOGFOOD**, **FOOBAR**, and **FOOBAR2**) |
| Development/Resources/Workstations | Contains individual developer workstations (e.g. **NIGHTCRAWLER** and **WOLVERINE**). |
| Development/Service Accounts | Contains service accounts for the Development environment (e.g. **Service account for SQL Server (DEV)** -- **TECHTOOLBOX\svc-sql-dev**) |
| Development/Users | Contains user accounts for each person in the Development organization (e.g. **TECHTOOLBOX\jjameson**) |
| IT | Represents the corporate "IT" group (which is really just my alter ego -- **TECHTOOLBOX\jjameson-admin**) that assumes all responsibility for managing the "Production" environment at Technology Toolbox |
| IT/Admin Accounts | Contains all user accounts that are members of the **Domain Admins** group. Note that all of these accounts are suffixed with "-admin" in order to easily distinguish them, as well as provide alternate "elevated privilege" accounts (e.g. **TECHTOOLBOX\jjameson-admin**) corresponding to low-privilege accounts (e.g. **TECHTOOLBOX\jjameson**) |
| IT/Groups | Contains the groups specific to the IT organization. |
| IT/Resources | Container for various resource OUs |
| IT/Resources/Servers | Contains servers for the "Test" and "Production" environments. Currently this include **BANSHEE**, **BEAST**, **COLOSSUS**, **CYCLOPS**, **DAZZLER**, **ICEMAN**, **JUBILEE**, and **ROGUE**. |
| IT/Resources/Workstations | Contains individual workstations for the IT organization. This OU is currently empty. |
| IT/Service Accounts | Contains service accounts for the "Test" and "Production" environments (e.g. **Service account for SQL Server (TEST) -- TECHTOOLBOX\svc-sql-test** and **Service account for SQL Server** -- **TECHTOOLBOX\svc-sql**) |
| IT/Users | Contains user accounts for each person in the IT organization. This OU is currently empty. |
| Sales | Represents the Sales organization within Technology Toolbox. I created this OU primarily for testing (for example, to verify whether I've configured the permissions correctly on a SharePoint site). |

{{< /table >}}

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
