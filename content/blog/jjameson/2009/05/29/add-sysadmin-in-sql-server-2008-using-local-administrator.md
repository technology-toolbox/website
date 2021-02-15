---
title: "Add sysadmin in SQL Server 2008 Using Local Administrator"
date: 2009-05-29T01:44:00-07:00
excerpt: "A couple of months ago, I had to SysPrep a copy of one of my VMs in order to remove dependencies on my home domain (I had to work out of the Microsoft office for a couple of days because my DSL router cratered). 
 However, while the SysPrep process was..."
draft: true
categories: [""]
tags: ["SQL Server"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/05/29/add-sysadmin-in-sql-server-2008-using-local-administrator.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/05/29/add-sysadmin-in-sql-server-2008-using-local-administrator.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

A couple of months ago, I had to SysPrep a copy of one of my VMs in order to remove dependencies on my home domain (I had to work out of the Microsoft office for a couple of days because my DSL router cratered).

However, while the SysPrep process was quick and painless, after the reboot I found that I could no longer use my domain account to administer SQL Server. Sure, I had local Administrator access, but starting with SQL Server 2005, being a local Admin no longer grants you seamless and unlimited access to SQL Server.

So the question that I had was: How do I add a user to the **sysadmin** role in SQL Server 2008 if all I have is local Administrator access?

Based on my experience with SQL Server 2005, I remembered this was done as part of the installation using the Surface Area Configuration wizard. However, I couldn't find the equivalent in SQL Server 2008. The SQL Server Books Online clearly stated that the Surface Area Configuration tool was discontinued, but the "Replacement" information didn't help me in the slightest.

I tried an Internet search (using both Windows Live Search and Google) for:

> add sysadmin SQL Server 2008

However, neither search returned anything useful (at least not within the first two pages of results -- which is typically my threshold before I move on to a different approach to solving a problem).

Fortunately for me, I didn't have to wait long for a response to my inquiry to one of the internal discussion lists. David Browne, a Technology Architect here at Microsoft, pointed me to the following:

<cite>Troubleshooting: Connecting to SQL Server When System Administrators Are Locked Out</cite>
[http://msdn.microsoft.com/en-us/library/dd207004.aspx](http://msdn.microsoft.com/en-us/library/dd207004.aspx)

I wish that would have come up in my search!

