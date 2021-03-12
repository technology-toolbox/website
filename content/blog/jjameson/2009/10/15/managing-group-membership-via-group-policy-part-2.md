---
title: "Managing Group Membership via Group Policy - Part 2"
date: 2009-10-15T05:04:00-06:00
excerpt: "In Part 1 of this post , I explained the Group Policy object (named Development - Restricted Groups Policy ) that I use for enforcing group membership on a specific set of servers. As a follow-up to that post, I also want to cover an alternate method..."
aliases: ["/blog/jjameson/archive/2009/10/14/managing-group-membership-via-group-policy-part-2.aspx", "/blog/jjameson/archive/2009/10/15/managing-group-membership-via-group-policy-part-2.aspx"]
draft: true
categories: ["My System", "Infrastructure"]
tags: ["My System", "Simplify", "Windows Server", "Infrastructure"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/15/managing-group-membership-via-group-policy-part-2.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/15/managing-group-membership-via-group-policy-part-2.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

In
[Part 1 of this post](/blog/jjameson/2009/10/15/managing-group-membership-via-group-policy-part-1),
I explained the Group Policy object (named **Development - Restricted Groups
Policy**) that I use for enforcing group membership on a specific set of
servers. As a follow-up to that post, I also want to cover an alternate method
of managing group membership.

In the previous scenario -- i.e. ensuring that Development team leads always
have administrative access to servers in their Development Integration
Environment (DEV) -- we actually wanted to restrict the members of the local
**Administrators** group on all servers in DEV. However, what if we need to
address a slightly different scenario in which we want a specific user or group
to always be a member of the local **Administrators** group -- in addition to
other group members (that vary by server)?

For example, consider the fact that I use
[Systems Center Operations Manager (SCOM)](http://www.microsoft.com/systemcenter/operationsmanager/en/us/default.aspx)
in order to monitor the various physical and virtual servers in the the
["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) (a.k.a.
my home lab). One of the things I learned while deploying SCOM is that it is,
um, *challenging* to deploy it in a least privilege configuration -- or at least
for someone who primarily considers himself an AppDev (Application Development)
flavor of Microsoft consultant.

At a bare minimum, your SCOM service account needs to be a member of the
**Performance Monitor Users** group on each monitored server. Rather than
forcing myself to configure this on all of my existing servers as well on new
servers and VMs that I will undoubtedly add in the future, I decided to apply
this change using Group Policy instead.

However, in this scenario, I don't want to *restrict* the members of the
**Performance Monitor Users** group on each monitored server. Rather I simply
want to ensure that the SCOM service account is a member of this group *in
addition to any other members*.

To address this scenario, I created a startup script called
**EnsureLocalGroupMembership.cmd** in the following folder:

> \\corp.technologytoolbox.com\SysVol\corp.technologytoolbox.com\Policies\{GUID}\Machine\Scripts\Startup\OperationsManager

The contents of the script are actually quite trival:

```
net localgroup "Performance Monitor Users" TECHTOOLBOX\svc-mom-action /add
```

> **Note**
>
> Prior to deploying SCOM 2007 in the "Jameson Datacenter" I used its
> predecessor -- Microsoft Operations Manager (MOM) -- and thus had already
> created a service account named **svc-mom-action**.

To force this startup script to run on all monitored servers, I created a Group
Policy object (named **Default Operations Manager Policy**) and linked it to the
corresponding OU.

Here are the settings for the Group Policy:

- **Computer Configuration**
  - **Policies**
    - **Windows Settings**
      - **Scripts (Startup/Shutdown)**
        - **Startup**
          - **Name: OperationsManager\EnsureLocalGroupMembership.cmd**

By linking this Group Policy to the appropriate OU (i.e.
**IT/Resources/Servers**) the SCOM service account is ensured to be a member of
the local **Performance Monitor Users** group on each monitored server. Voilà!

