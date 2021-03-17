---
title: "\"Web application at ... could not be found\" Error on MOSS 2007 x64"
date: 2009-10-08T08:49:00-06:00
excerpt:
  "I encountered a rather nasty bug last week with Microsoft Office SharePoint
  Server (MOSS) 2007 when trying to run an x86 process (that utilizes the
  SharePoint API) on an x64 server. 
   To provide the simplest repro possible, I created a sample console..."
aliases: ["/blog/jjameson/archive/2009/10/07/web-application-at-could-not-be-found-error-on-moss-2007-x64.aspx", "/blog/jjameson/archive/2009/10/08/web-application-at-could-not-be-found-error-on-moss-2007-x64.aspx"]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/08/web-application-at-could-not-be-found-error-on-moss-2007-x64.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/08/web-application-at-could-not-be-found-error-on-moss-2007-x64.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

I encountered a rather nasty bug last week with Microsoft Office SharePoint
Server (MOSS) 2007 when trying to run an x86 process (that utilizes the
SharePoint API) on an x64 server.

To provide the simplest repro possible, I created a sample console application
that prints the title of a SharePoint site:

```
using System;

using Microsoft.SharePoint;

namespace Fabrikam.Demo.PrintSharePointSiteTitle
{
    class Program
    {
        static void Main(
            string[] args)
        {
            if (args.Length < 1)
            {
                Console.Error.WriteLine(
                    "Usage: PrintSharePointSiteTitle {site URL}");

                Environment.Exit(1);
            }

            string siteUrl = args[0];

            using (SPSite site = new SPSite(siteUrl))
            {
                Console.WriteLine(
                    "Title: {0}",
                    site.RootWeb.Title);
            }
        }
    }
}
```

Note that by default, Visual Studio projects set the platform target to **Any
CPU**. Thus when you run this program on an x64 server, the process runs
natively in 64-bit and everything works as expected:

{{< console-block-start >}}

C:\NotBackedUp\Temp\PrintSharePointSiteTitle\bin\Debug&gt;{{< kbd "PrintSharePointSiteTitle.exe http://fabrikam-local" >}}

```
{{< sample-output "Title: Fabrikam" >}}
```

{{< console-block-end >}}

However, if you change the platform target to **x86** and thus force the process
to run in 32-bit then things don't go well:

{{< console-block-start >}}

C:\NotBackedUp\Temp\PrintSharePointSiteTitle\bin\Debug&gt;{{< kbd "PrintSharePointSiteTitle.exe http://fabrikam-local" >}}

```
Unhandled Exception: System.IO.FileNotFoundException: The Web application at http://fabrikam-local
could not be found. Verify that you have typed the URL correctly. If the URL should
be serving existing content, the system administrator may need to add a new request
URL mapping to the intended application.
at Microsoft.SharePoint.SPSite..ctor(SPFarm farm, Uri requestUri, Boolean contextSite, SPUserToken userToken)
at Microsoft.SharePoint.SPSite..ctor(String requestUrl)
at Fabrikam.Demo.PrintSharePointSiteTitle.Program.Main(String[] args) in C:\NotBackedUp\Temp\PrintSharePointSiteTitle\Program.cs:line 22
```

{{< console-block-end >}}

I could certainly understand getting a
[`BadImageFormatException`](http://msdn.microsoft.com/en-us/library/system.badimageformatexception.aspx)
in this scenario (in other words, if the platform target for
Microsoft.SharePoint.dll was set to x64, then you shouldn't expect to be able to
load it from a 32-bit process). Instead I get a "bogus" `FileNotFoundException`
suggesting my Web application doesn't exist.

At this point, you're probably thinking something like "Jeremy, why would you
possibly want to run a 32-bit process on a 64-bit server?" Great question...

There are at least a couple of scenarios that I'm aware of:

- First, you might want to run some Visual Studio
  [unit tests that access a live SharePoint site](/blog/jjameson/2007/03/22/what-s-in-a-name-defaultfeaturereceiver-vs-featureconfigurator).
  These unit tests work great when you are developing on an x86 MOSS 2007 VM.
  However, they "blow chunks" on an x64 development VM. [Oh, and please don't
  get me started on the whole "mocking" topic for unit testing SharePoint. That
  could get me going for hours -- and I really do have more important things to
  do today ;-) ]
- Second, you might have written a utility, let's say, something like
  ImportPages.exe, that bulk loads pages into a SharePoint site using an Excel
  input file. Since the SharePoint API in MOSS 2007 is not "remoteable" (without
  switching to calling Web services), you must run ImportPages.exe directly on
  one of the SharePoint servers in the farm.

It's the latter scenario in which I first encountered this bug. However, rather
than going into the details here, I want to cover the ImportPages.exe utility in
a
[separate post](/blog/jjameson/2009/10/08/importing-pages-into-moss-2007-from-an-excel-file).
