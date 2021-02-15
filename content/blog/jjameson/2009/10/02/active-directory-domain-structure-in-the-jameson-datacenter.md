---
title: "Active Directory Domain Structure in the \"Jameson Datacenter\""
date: 2009-10-02T01:50:00-07:00
excerpt: "In a previous post, I provided some details on the \"Jameson Datacenter\" , which is really just my home lab that I use for learning new technologies and improving my skills, as well as actually completing my day-to-day tasks on various customer projects..."
draft: true
categories: ["My System", "Infrastructure"]
tags: ["My System", "Infrastructure"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/02/active-directory-domain-structure-in-the-jameson-datacenter.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/02/active-directory-domain-structure-in-the-jameson-datacenter.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

In a previous post, I provided some details on the ["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter), which is really just my home lab that I use for learning         new technologies and improving my skills, as well as actually completing my day-to-day         tasks on various customer projects (whenever I am actually able to work from home         instead of using my laptop at a customer site).

In this post, I want to share details about how I've configured my Active Directory         domain, namely **corp.technologytoolbox.com**.

> **Note**
>
> Technology Toolbox is just a name I came up with about ten years ago before joining Microsoft. At the time, I was considering starting my own company and therefore went looking for an available domain name. (I would have preferred techtoolbox.com, but that was already taken.) [Don't bother trying to browse to [https://www.technologytoolbox.com](https://www.technologytoolbox.com/) because, honestly, I haven't yet invested the time in actually creating the Web site -- even after almost ten years.]

As you can see in the following figure, I've created a number of organizational         units (OUs) which you would typically find in many enterprise organizations.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www%5Ftechnologytoolbox%5Fcom/blog/jjameson/8/r%5FJameson%20Datacenter%20-%20AD%20domain%20structure.png"
alt="Jameson Datacenter - Active Directory domain structure"
height="600"    width="302"
title="Figure 1: Jameson Datacenter - Active Directory domain structure" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/8/o_Jameson%20Datacenter%20-%20AD%20domain%20structure.png)

Note that I didn't really come up with this structure myself (except for the **Admin Accounts** OU, which I believe was an ["original idea"](http://en.wikipedia.org/wiki/A_Beautiful_Mind_%28film%29) of mine -- but honestly it's been so long that I can         hardly remember). Most of this is based on the prescriptive guidance from TechNet.         You can find the latest version of this guidance in the [AD DS (Active Directory Domain Services) Design Guide](http://technet.microsoft.com/en-us/library/cc754678%28WS.10%29.aspx). [I tried to find         the original reference material that I used, but I gave up looking for it on TechNet         this morning after about 5 minutes.]

As you can see, I created OUs to model organizations such as the **Development**         group, the **IT** group, and the **Sales** organization.

The following table summarizes the various OUs in the domain:

<caption>            Organizational Units in the corp.technologytoolbox.com Domain</caption>|                     Organizational Unit<br>                 |                     Description<br>                 |
| --- | --- |
|                     Development<br>                 |                     Represents the "Development" group within Technology Toolbox (which really just<br>                    means me, since my wife doesn't do any development and my 3-1/2 year old daughter<br>                    isn't quite ready for .NET yet)<br>                 |
|                     Development/Groups<br>                 |                     Contains the groups within the Development organization (such as **TECHTOOLBOX\All<br>                        Developers** and **TECHTOOLBOX\Development Admins**). Delegated<br>                    administration is used to allow developers to create and manage their own groups.<br>                 |
|                     Development/Resources<br>                 |                     Container for various resource OUs<br>                 |
|                     Development/Resources/Servers<br>                 |                     Contains servers for the Development environment (e.g. **DOGFOOD**,<br>                    **FOOBAR**, and **FOOBAR2**)<br>                 |
|                     Development/Resources/Workstations<br>                 |                     Contains individual developer workstations (e.g. **NIGHTCRAWLER** and<br>                    **WOLVERINE**).<br>                 |
|                     Development/Service Accounts<br>                 |                     Contains service accounts for the Development environment (e.g. **Service account<br>                        for SQL Server (DEV)** -- **TECHTOOLBOX\svc-sql-dev**)<br>                 |
|                     Development/Users<br>                 |                     Contains user accounts for each person in the Development organization (e.g. **TECHTOOLBOX\jjameson**)<br>                 |
|                     IT<br>                 |                     Represents the corporate "IT" group (which is really just my alter ego -- **TECHTOOLBOX\jjameson-admin**)<br>                    that assumes all responsibility for managing the "Production" environment at Technology<br>                    Toolbox<br>                 |
|                     IT/Admin Accounts<br>                 |                     Contains all user accounts that are members of the **Domain Admins**<br>                    group. Note that all of these accounts are suffixed with "-admin" in order to easily<br>                    distinguish them, as well as provide alternate "elevated privilege" accounts (e.g.<br>                    **TECHTOOLBOX\jjameson-admin**) corresponding to low-privilege accounts<br>                    (e.g. **TECHTOOLBOX\jjameson**)<br>                 |
|                     IT/Groups<br>                 |                     Contains the groups specific to the IT organization.<br>                 |
|                     IT/Resources<br>                 |                     Container for various resource OUs<br>                 |
|                     IT/Resources/Servers<br>                 |                     Contains servers for the "Test" and "Production" environments. Currently this include<br>                    **BANSHEE**, **BEAST**, **COLOSSUS**, **CYCLOPS**, **DAZZLER**, **ICEMAN**, **JUBILEE**,<br>                    and **ROGUE**.<br>                 |
|                     IT/Resources/Workstations<br>                 |                     Contains individual workstations for the IT organization. This OU is currently empty.<br>                 |
|                     IT/Service Accounts<br>                 |                     Contains service accounts for the "Test" and "Production" environments (e.g. **Service account for SQL Server (TEST) -- TECHTOOLBOX\svc-sql-test** and<br>                    **Service account for SQL Server** -- **TECHTOOLBOX\svc-sql**)<br>                 |
|                     IT/Users<br>                 |                     Contains user accounts for each person in the IT organization. This OU is currently<br>                    empty.<br>                 |
|                     Sales<br>                 |                     Represents the Sales organization within Technology Toolbox. I created this OU primarily<br>                    for testing (for example, to verify whether I've configured the permissions correctly<br>                    on a SharePoint site).<br>                 |
> **Note**
>
> The **Sales** OU contains similar OUs as **Development**. These are simply not shown in the previous table.

Note that this domain structure, I'm able to use the Group Policy feature of Active         Directory to "effortlessly" configure new servers.

For example, this morning I created a new development VM (FOOBAR3) from a [SysPrep'ed VHD](/blog/jjameson/2009/08/13/using-sysprep-ed-vhds-for-new-hyper-v-virtual-machines) file. After joining the new VM to the domain, I then moved         FOOBAR3 to the **Development/Resources/Servers** OU. After the VM rebooted         (which is required when you join a domain), it was automatically configured with         numerous settings, including Windows Update (i.e. to update from my WSUS server         -- COLOSSUS) and also to grant the **TECHTOOLBOX\Development Admins**         group administrative access (of which **TECHTOOLBOX\jjameson** is a         member). I was then able to quickly log in with my regular account and start installing         SQL Server 2008 (which finished before I actually completed this post). Now it's         time to install Visual Studio 2008...

I'll cover my group policy configuration in a separate post.

