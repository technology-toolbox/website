---
title: BackedUp and NotBackedUp
date: 2007-03-22T03:59:00-06:00
excerpt:
  About four years ago, one of the partners that I was working on an engagement
  with commented on how developers tend to have their own unique way of managing
  files, but that mine was one of the most bizarre he had ever seen. It has been
  four years, but...
aliases:
  [
    "/blog/jjameson/archive/2007/03/21/backedup-and-notbackedup.aspx",
    "/blog/jjameson/archive/2007/03/22/backedup-and-notbackedup.aspx",
  ]
draft: true
categories: ["My System"]
tags: ["My System", "Toolbox"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/03/22/backedup-and-notbackedup.aspx"
---

About four years ago, one of the partners that I was working on an engagement
with commented on how developers tend to have their own unique way of managing
files, but that mine was one of the most bizarre he had ever seen. It has been
four years, but I believe the statement [Ted](http://weblogs.asp.net/tgraham)
made was something along the lines of

> "...and Jeremy puts all his files in only two folders"

Well, truth be told, Ted was right -- and he would still be right if he repeated
the statement today.

On all of my various laptops, desktops, and servers, you will find that the only
folders in the root of any drive are the ones created by Windows (such as
Documents and Settings, Inetpub, Program Files, and Windows) as well as the two
that I create:

- BackedUp
- NotBackedUp

I tend to put files that I create -- or contribute to -- in the **BackedUp**
folder. Everything else I put in the **NotBackedUp** folder. Why, you may ask,
do I use this apparently over-simplistic system?

I attribute the reasoning to several factors:

1. I download a lot of stuff (i.e. gigabytes) , much of which I need to access
   frequently -- regardless of which particular computer I happen to be working
   on at any particular moment
1. If I were to lose any of the files that I create -- or contribute to -- it
   could require several days (or actually several late nights) to recreate the
   material; some of the documents that I have created have several man-weeks
   invested in them -- I dread the mere thought of losing one of those
1. Long before the user experience improvements in Windows Vista, I needed
   "instant access" to my frequently used files (in fact, I still need these
   shortcuts whenever I am logged into a computer or VM running Windows Server
   2003)
1. I typically "rebuild" my laptop once a year either to install a new OS (such
   as last November when Vista was released) or to clean out all the old garbage
   that, being a developer, I tend to install for one reason or another

Factor #1 is addressed by creating various folders under NotBackedUp and storing
the gigabytes of stuff that I have managed to download over the last 8 years in
various folders underneath:

> \NotBackedUp\Builds\
> ...\
> \NotBackedUp\Public\Download\
> \NotBackedUp\Public\Toolbox\
> ...\
> \NotBackedUp\Temp\
> \NotBackedUp\VMs

Factor #2 is addressed by redirecting the "My Documents" folder to

> \BackedUp\jjameson\Documents

...and then setting up a simple batch file to copy everything in the BackedUp
folder to one of my home servers:

{{< console-block-start >}}

robocopy C:\BackedUp \\beast\Backups\jjameson1\BackedUp /E

{{< console-block-end >}}

Factor #3 is simply a matter of creating two shortcuts on my taskbar:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/My-System/Taskbar-Shortcuts-301x96.jpg"
alt="Taskbar Shortcuts" height="96" width="301"
title="Figure 1 - Taskbar Shortcuts" >}}

Factor #4 (rebuilding my laptop) certainly isn't a quick ordeal (it typically
takes about 4 hours, by the time I install the OS, SQL Server, Visual Studio,
copy my gigabytes worth of downloads back from the server, etc.) but at least I
don't have to worry about accidentally losing any of my files. I just do a quick
robocopy of the BackedUp and NotBackedUp folders to one of my home servers, pop
in the Windows installation CD, and format my partition.

By limiting the number of places that I store files, I dramatically simplify the
management of my files.

I'd like to tell you that the concept of BackedUp and NotBackedUp was something
that I came up with, but actually it's something I picked up from a Unix
sysadmin by the name of Gary Rittenhouse, who I used to work with at AT&T Bell
Laboratories. [Gary, you'll probably never see this post, but if you do, thanks
for all of the great tips and tricks you provided me in those early days.]
