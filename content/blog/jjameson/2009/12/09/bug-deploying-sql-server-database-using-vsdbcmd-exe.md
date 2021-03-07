---
title: "Bug Deploying SQL Server Database Using VSDBCMD.EXE"
date: 2009-12-09T06:32:00-07:00
excerpt: "Yesterday we encountered a bug while trying to deploy a new SQL Server database from a Visual Studio database project using the VSDBCMD.EXE utility, following the prescriptive guidance on MSDN: 
 How to: Prepare a Database for Deployment From a Command..."
aliases: ["/blog/jjameson/archive/2009/12/08/bug-deploying-sql-server-database-using-vsdbcmd-exe.aspx", "/blog/jjameson/archive/2009/12/09/bug-deploying-sql-server-database-using-vsdbcmd-exe.aspx"]
draft: true
categories: ["Development"]
tags: ["SQL Server", "Visual Studio"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/12/09/bug-deploying-sql-server-database-using-vsdbcmd-exe.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/12/09/bug-deploying-sql-server-database-using-vsdbcmd-exe.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Yesterday we encountered a bug while trying to deploy a new SQL Server database
from a Visual Studio database project using the VSDBCMD.EXE utility, following
the prescriptive guidance on MSDN:

{{< reference title="How to: Prepare a Database for Deployment From a Command Prompt by Using VSDBCMD" linkHref="http://msdn.microsoft.com/en-us/library/dd193258.aspx" >}}

According to this MSDN article, all you need to do is copy some files to the
server running Microsoft SQL Server, and then run the VSDBCMD.EXE utility
(specifying your .dbschema or .deploymanifest file).

Unfortunately, when we attempted to deploy our database to our Test environment,
we encountered a `NullReferenceException`:

{{< blockquote "font-italic text-danger" >}}

Object reference not set to an instance of an object.

{{< /blockquote >}}

Fortunately, it didn't take long to find
[Gert's recommendation](http://social.msdn.microsoft.com/Forums/en-US/vstsdb/thread/32725cf6-74c1-4b5a-9057-b909ae8a2517)
to add the following registry key:

{{< console-block-start >}}

reg add HKEY\_CURRENT\_USER\Software\Microsoft\VisualStudio\9.0

{{< console-block-end >}}

The error occurred since Visual Studio has never been installed on the SQL
Server in TEST (unlike our local development VMs).

It certainly would be nice if the above MSDN article were updated to note the
bug and associated workaround.

> **Note**
>
> As pointed out in a comment by Ramkumar Perumal, Microsoft.SqlServer.BatchParser.dll does not exist in any of the specified folders. This should also be reflected in the MSDN article.

You will also find instructions to add the registry key -- as well as an
explanation for why you more than likely already have
Microsoft.SqlServer.BatchParser.dll installed on your SQL Server -- in the
following blog post:

{{< reference title="Deploying your Database Project without VSTSDB installed. 2009-02-21" linkHref="http://blogs.msdn.com/bahill/archive/2009/02/21/deploying-your-database-project-without-vstsdb-installed.aspx" >}}

