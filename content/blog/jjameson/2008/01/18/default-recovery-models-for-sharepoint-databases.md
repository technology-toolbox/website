---
title: "Default Recovery Models for SharePoint Databases"
date: 2008-01-18T09:59:00-07:00
excerpt: "Okay, I haven't blogged in over 7 weeks -- but hey, I was on vacation for 3 of them -- and I must warn you upfront that this post isn't exactly a \"zinger\" filled with juicy tidbits, recommendations, or workarounds. Rather, I simply can't seem to remember..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2008/01/18/default-recovery-models-for-sharepoint-databases.aspx](http://blogs.msdn.com/b/jjameson/archive/2008/01/18/default-recovery-models-for-sharepoint-databases.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

Okay, I haven't blogged in over 7 weeks -- but hey, I was on vacation for 3 of  them -- and I must warn you upfront that this post isn't exactly a "zinger" filled  with juicy tidbits, recommendations, or workarounds. Rather, I simply can't seem  to remember what Microsoft Office SharePoint Server (MOSS) 2007 sets the recovery  model to for the various databases it creates. Most of the time you don't really  care -- but around 1:00 AM this morning I had just wrapped up completely rebuilding  our Development environment (DEV) to cleanup the "junk" that tends to accumulate  over time as various developers work on their respective features and deploy and  configure them in an integrated environment. Now it's time to start migrating some  content into the freshly created sites. However, before I start running the content  migration utilities...

The primary reason that I care about the default recovery model is so that for  DEV, I can quickly change the recovery model of the SharePoint databases to Simple  so that the transaction log gets truncated on checkpoint. In other words, for a  development environment, you are typically not concerned with being able to recover  data up to some particular point. Rather, my primary concern is to minimize the  amount of disk space consumed (since DEV is often a virtual environment) and, perhaps  even more important, minimize the amount of administration that needs to be done  to keep DEV in a "healthy" state (meaning that it is available and serving the needs  of the team). Also, if DEV were to actually crash and somehow corrupt the data,  it is highly unlikely that I'd even attempt to do some kind of restore. Rather,  I tend to just invest some hours rebuilding DEV -- preferably using as many scripts  as possible to automate the mundane tasks of creating Web Applications and site  collections, adding and deploying solutions, activating features, etc.

Besides, what's that line from Aliens..."I say we nuke the site from orbit...it's  the only way to be sure." I love that line. What better way to cleanup all the "junk"  and ensure you have a documented, repeatable process for deploying your SharePoint  solutions, than periodically nuking the site and starting from a "known state" that  matches TEST and PROD?

Anyway, back to the default recovery models...

From our freshly rebuilt DEV SharePoint environment, here are the databases and  the various default recovery models:

| Database Name | Default Recovery Model |
| --- | --- |
| SharePoint\_AdminContent\_{GUID} | Full |
| SharePoint\_Config | Full |
| {SSP name}\_DB | Simple |
| {SSP name}\_Search\_DB | Simple |
| WSS\_Content | Full |
| WSS\_Content\_{SSP name} | Full |
| WSS\_Search\_{server name} | Simple |

Well, there you have it. Like I warned you at the outset, it's not very "juicy",  but at least I can refer back here from time to time as my memory fades.

Oops...almost forgot..here's some SQL to quickly toggle the recovery model for  a database:

```
USE [master]
GO
ALTER DATABASE [WSS_Content] SET RECOVERY SIMPLE
GO
ALTER DATABASE [WSS_Content] SET RECOVERY FULL
GO
```

Again, probably more for my benefit, than yours ;-)
