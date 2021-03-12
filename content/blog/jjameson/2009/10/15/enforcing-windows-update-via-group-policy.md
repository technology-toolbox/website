---
title: "Enforcing Windows Update via Group Policy"
date: 2009-10-15T04:15:00-06:00
excerpt: "Another Group Policy object that I use in the \"Jameson Datacenter\" (a.k.a. my home lab) is one to automatically configure Windows Update on all computers in the domain. This ensures that each server or workstation downloads updates from COLOSSUS (one..."
aliases: ["/blog/jjameson/archive/2009/10/14/enforcing-windows-update-via-group-policy.aspx", "/blog/jjameson/archive/2009/10/15/enforcing-windows-update-via-group-policy.aspx"]
draft: true
categories: ["My System", "Infrastructure"]
tags: ["My System", "Simplify", "WSUS", "Windows Server", "Infrastructure"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/15/enforcing-windows-update-via-group-policy.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/15/enforcing-windows-update-via-group-policy.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Another Group Policy object that I use in the
["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) (a.k.a.
my home lab) is one to automatically configure Windows Update on all computers
in the domain. This ensures that each server or workstation downloads updates
from COLOSSUS (one of my VMs that is running Windows Server Update Services)
rather than having each computer download, for example, a
[577 MB service pack](http://www.microsoft.com/downloads/details.aspx?FamilyID=656c9d4a-55ec-4972-a0d7-b1a6fedf51a7&displaylang=en)
directly from the Internet. It also ensures that only the updates that I have
approved through WSUS are applied.

To automatically configure Windows Update in the "Jameson Datacenter", I have
defined a Group Policy (named **Default Windows Update Policy**) with the
following settings:

- **Computer Configuration**
  - **Policies**
    - **Administrative Templates**
      - **Windows Components**
        - **Windows Update**
          - **Configure Automatic Updates**
            - **Enabled**
            - **Configure automatic updating: 4 -Auto download and schedule the
              install**
            - **Scheduled install day: 0 - Every day**
            - **Scheduled install time: 03:00**
          - **Specify intranet Microsoft update service location**
            - **Enabled**
            - **Set the intranet update service for detecting updates:
              http://colossus**
            - **Set the intranet statistics server: http://colossus**

By linking this Group Policy to the entire domain (i.e.
**corp.technologytoolbox.com**) Windows Update is automatically configured as
soon as new computers are joined to the domain and rebooted.

This enables me to spin up new VMs with very little effort. More importantly, it
takes less than a half hour to get a new Windows Server 2008 VM with all the
latest patches (since I start from a
[SysPrep'ed VHD](/blog/jjameson/2009/08/13/using-sysprep-ed-vhds-for-new-hyper-v-virtual-machines)
with Windows Server 2008 Service Pack 2).

