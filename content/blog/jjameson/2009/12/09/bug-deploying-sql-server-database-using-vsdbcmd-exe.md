---
title: Bug Deploying SQL Server Database Using VSDBCMD.EXE
date: 2009-12-09T06:32:00-07:00
description:
  "Yesterday we encountered a bug while trying to deploy a new SQL Server
  database from a Visual Studio database project using the VSDBCMD.EXE utility,
  following the prescriptive guidance on MSDN: How to: Prepare a Database for
  Deployment From a Command..."
aliases:
  [
    "/blog/jjameson/archive/2009/12/08/bug-deploying-sql-server-database-using-vsdbcmd-exe.aspx",
    "/blog/jjameson/archive/2009/12/09/bug-deploying-sql-server-database-using-vsdbcmd-exe.aspx",
  ]
categories: ["Development"]
tags: ["SQL Server", "Visual Studio"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/12/09/bug-deploying-sql-server-database-using-vsdbcmd-exe.aspx"
---

Yesterday we encountered a bug while trying to deploy a new SQL Server database
from a Visual Studio database project using the VSDBCMD.EXE utility, following
the prescriptive guidance on MSDN:

{{< reference
title="How to: Prepare a Database for Deployment From a Command Prompt by Using VSDBCMD"
linkHref="http://msdn.microsoft.com/en-us/library/dd193258.aspx" >}}

According to this MSDN article, all you need to do is copy some files to the
server running Microsoft SQL Server, and then run the VSDBCMD.EXE utility
(specifying your .dbschema or .deploymanifest file).

Unfortunately, when we attempted to deploy our database to our Test environment,
we encountered a `NullReferenceException`:

{{< div-block "errorMessage" >}}

> Object reference not set to an instance of an object.

{{< /div-block >}}

Fortunately, it didn't take long to find
[Gert's recommendation](http://social.msdn.microsoft.com/Forums/en-US/vstsdb/thread/32725cf6-74c1-4b5a-9057-b909ae8a2517)
to add the following registry key:

{{< console-block >}}

reg add HKEY_CURRENT_USER\Software\Microsoft\VisualStudio\9.0

{{< /console-block >}}

The error occurred since Visual Studio has never been installed on the SQL
Server in TEST (unlike our local development VMs).

It certainly would be nice if the above MSDN article were updated to note the
bug and associated workaround.

{{< div-block "note" >}}

> **Note**
>
> As pointed out in a comment by Ramkumar Perumal,
> Microsoft.SqlServer.BatchParser.dll does not exist in any of the specified
> folders. This should also be reflected in the MSDN article.

{{< /div-block >}}

You will also find instructions to add the registry key -- as well as an
explanation for why you more than likely already have
Microsoft.SqlServer.BatchParser.dll installed on your SQL Server -- in the
following blog post:

{{< reference
title="Deploying your Database Project without VSTSDB installed. 2009-02-21"
linkHref="http://blogs.msdn.com/bahill/archive/2009/02/21/deploying-your-database-project-without-vstsdb-installed.aspx" >}}
