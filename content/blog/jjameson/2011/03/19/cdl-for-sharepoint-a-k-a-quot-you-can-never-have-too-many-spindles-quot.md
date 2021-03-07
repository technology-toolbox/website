---
title: "CDL for SharePoint (a.k.a. \"You can never have too many spindles\")"
date: 2011-03-19T04:55:00-06:00
excerpt: "In the United States, \"CDL\" typically refers to a Commercial Driver's License -- but since I don't drive trucks for a living, I use the acronym for something entirely different. To me, these three letters correspond to the minimum number of drives I like..."
aliases: ["/blog/jjameson/archive/2011/03/18/cdl-for-sharepoint-a-k-a-quot-you-can-never-have-too-many-spindles-quot.aspx", "/blog/jjameson/archive/2011/03/19/cdl-for-sharepoint-a-k-a-quot-you-can-never-have-too-many-spindles-quot.aspx"]
draft: true
categories: ["My System", "SharePoint", "Infrastructure"]
tags: ["My System", "MOSS 2007", "Infrastructure", "SharePoint 2010"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/03/19/cdl-for-sharepoint-a-k-a-quot-you-can-never-have-too-many-spindles-quot.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/03/19/cdl-for-sharepoint-a-k-a-quot-you-can-never-have-too-many-spindles-quot.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

In the United States, "CDL" typically refers to a Commercial Driver's License --
but since I don't drive trucks for a living, I use the acronym for something
entirely different. To me, these three letters correspond to the minimum number
of drives I like to see on any server running SharePoint Server 2010 -- or its
predecessor, Microsoft Office SharePoint Server 2007.

In other words, I think *all* SharePoint servers -- yes, even your development
VMs -- should have a C: drive, a D: drive, and an L: drive.

In general, the C: drive is used for operating system files, program files, the
paging file, and the IIS logs. The D: drive (which I typically label "Data01")
and L: drive ("Log01") are used differently depending on the environment.

In a Production environment, the D: drive is used for the search index files
(and possibly the BlobCache too, depending on your Search load) and the L: drive
is used for the SharePoint ULS logs.

You certainly don't have to separate the search index files and ULS logs onto
separate disks. If you do, then your SharePoint servers should be able to scale
"massively" (provided you have sufficient CPU and memory). If you don't isolate
the search index files and ULS logs, then personally I wouldn't use "massive" to
describe the scalability of your SharePoint architecture, because I believe you
are likely to bottleneck on disk I/O on the front-end Web/query servers long
before you max out today's blazingly fast, mulit-core processors and
high-capacity DIMMs.

On development VMs, I create separate VHDs for the D: and L: drives for several
reasons:

- Unlike Production environments, SQL Server is typically installed "side-by-side" with SharePoint in development environments. Thus I make sure that when installing SQL Server I change the default locations of database files and log files. Personally, I like to use "[D:\NotBackedUp\Microsoft SQL Server\MSSQL10.MSSQLSERVER\MSSQL\DATA](file:///D:/NotBackedUp/Microsoft%20SQL%20Server/MSSQL10.MSSQLSERVER/MSSQL/DATA)" and "[L:\NotBackedUp\Microsoft SQL Server\MSSQL10.MSSQLSERVER\MSSQL\DATA](file:///L:/NotBackedUp/Microsoft%20SQL%20Server/MSSQL10.MSSQLSERVER/MSSQL/DATA)").
- On my Hyper-V server (ICEMAN) that I use to run SharePoint development VMs, I have two RAID 1 arrays and therefore split the VHDs across the two drives on the host, thus greatly reducing the possibility of I/O contention on a single physical drive.
- Whenever I need to run a SharePoint VM from my laptop, I can easily split the VHDs between the laptop's internal hard drive and an external USB drive. [Note that I haven't run a SharePoint VM on my laptop since replacing the internal drive with a solid-state drive (SSD) so this may no longer be an issue. However, back when I had to do this at customer sites, using a combination of the internal drive and an external USB drive was the only way I could keep my sanity when running SharePoint VMs.]

When I used to have ICEMAN configured with a single logical drive (four physical
drives in a RAID 10 array), I would occasionally receive alerts from Operations
Manager regarding high disk latency. Since splitting those same physical drives
into two RAID 1 arrays, I haven't received any more disk latency alerts.

