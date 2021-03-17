---
title: "Managing Group Membership via Group Policy - Part 1"
date: 2009-10-15T03:45:00-06:00
excerpt: "In yesterday's post I covered one of the Group Policy objects that I use in the \"Jameson Datacenter\" (a.k.a. my home lab), specifically one that automatically enables Remote Desktop (Terminal Services) whenever I add a new server to my Active Directory..."
aliases: ["/blog/jjameson/archive/2009/10/14/managing-group-membership-via-group-policy-part-1.aspx", "/blog/jjameson/archive/2009/10/15/managing-group-membership-via-group-policy-part-1.aspx"]
draft: true
categories: ["My System", "Infrastructure"]
tags: ["My System", "Simplify", "Windows Server", "Infrastructure"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/15/managing-group-membership-via-group-policy-part-1.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/15/managing-group-membership-via-group-policy-part-1.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

In yesterday's
[post](/blog/jjameson/2009/10/14/enabling-remote-desktop-via-group-policy) I
covered one of the Group Policy objects that I use in the
["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) (a.k.a.
my home lab), specifically one that automatically enables Remote Desktop
(Terminal Services) whenever I add a new server to my Active Directory domain.
This post introduces a Group Policy object that enforces group membership on a
specific set of servers.

To understand the value of this kind of Group Policy, consider a scenario where
you have a Development organization that manages its own
[Development Integration Environment (DEV)](/blog/jjameson/2009/09/25/development-and-build-environments).
Standard operating procedures state that certain individuals within the
Development organization -- say, for example, the team leads -- are given full
administrative access to the servers in this environment. These individuals are
members of the
[**Development Admins**](/blog/jjameson/2009/10/02/active-directory-domain-structure-in-the-jameson-datacenter)
domain group. In order to avoid having to explicitly add this domain group to
the local **Administrators** group on each server in DEV, you can instead manage
the group membership through Group Policy. Thus, whenever the Development team
"spins up" a new server for their environment, all of the Development team leads
are granted administrative access as soon as the server is joined to the domain.

To address this scenario in the "Jameson Datacenter", I have defined a Group
Policy (named **Development - Restricted Groups Policy**) with the following
settings:

- **Computer Configuration**
  - **Policies**
    - **Windows Settings**
      - **Security Settings**
        - **Restricted Groups**
          - **Group Name: Administrators**
          - **Members: TECHTOOLBOX\Development Admins, TECHTOOLBOX\Domain
            Admins**

By linking this Group Policy to the appropriate OU (i.e.
**Development/Resources/Servers**) the members of the local **Administrators**
group are automatically configured as soon as I join a new DEV server to the
domain and reboot.

See
[Part 2 of this post](/blog/jjameson/2009/10/15/managing-group-membership-via-group-policy-part-2)for
an alternate method of managing group membership through Group Policy.
