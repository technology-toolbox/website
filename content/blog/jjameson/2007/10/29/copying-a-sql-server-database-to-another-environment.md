---
title: Copying a SQL Server Database to Another Environment
date: 2007-10-29T08:03:00-06:00
description:
  A couple of weeks ago I was troubleshooting a performance problem with the
  variations feature in Microsoft Office SharePoint Server (MOSS) 2007 and I
  needed to copy the content database to another environment for further
  analysis and testing. An easy...
aliases:
  [
    "/blog/jjameson/archive/2007/10/28/copying-a-sql-server-database-to-another-environment.aspx",
    "/blog/jjameson/archive/2007/10/29/copying-a-sql-server-database-to-another-environment.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007", "SQL Server"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/10/29/copying-a-sql-server-database-to-another-environment.aspx"
---

A couple of weeks ago I was troubleshooting a performance problem with the
variations feature in Microsoft Office SharePoint Server (MOSS) 2007 and I
needed to copy the content database to another environment for further analysis
and testing. An easy (an unobtrusive) way to "snapshot" a database and copy it
to another environment is to create a backup with the **COPY_ONLY** option:

```SQL
BACKUP DATABASE [WSS_Content]
TO DISK = N'H:\WSS_Content.bak'
WITH NOFORMAT, NOINIT
    , NAME = N'WSS_Content-Full Database Backup'
    , SKIP, NOREWIND, NOUNLOAD, STATS = 10
    , COPY_ONLY
```

From SQL Server 2005 Books Online:

{{< div-block "fst-italic" >}}

> Taking a backup normally changes the database, in turn affecting other backups
> and how they are restored. Sometimes, however, a backup must be taken for a
> special purpose that should not affect the overall backup and restore
> procedures for the database.
>
> A data backup is normally a base backup for one or more differential backups
> taken after it. Microsoft SQL Server 2005 introduces support for creating
> copy-only backups, which do not affect the normal sequence of backups.
> Therefore, unlike other backups, a copy-only backup does not impact the
> overall backup and restore procedures for the database.

{{< /div-block >}}

In other words, by using the **COPY_ONLY** option I avoided screwing up the
scheduled differential backups on the database.

However, there are a couple of issues with this approach:

- You cannot specify the **COPY_ONLY** option through the UI in SQL Server
  Management Studio, but this is no big deal -- you can start by configuring
  most of the backup options using the UI, script the action to generate the
  corresponding SQL, and then add the **COPY_ONLY** option as shown above
- You cannot restore a backup created using the **COPY_ONLY** option through the
  UI in SQL Server Management Studio; in the **Restore Database** dialog, when
  you select the **From device** option and then specify the backup file
  previously created with the **COPY_ONLY** option, no backup sets are displayed

The second problem was puzzling to me. After specifying my backup file, when I
attempted to change to the **Options** page, I encountered the following error:

{{< div-block "errorMessage" >}}

> You must select a restore source.

{{< /div-block >}}

When I first encountered this problem, I thought I had a corrupt backup file.
However, by once again reverting to SQL instead of the UI, I was able to verify
the backup was, in fact, valid:

```SQL
RESTORE FILELISTONLY
FROM DISK = N'E:\NotBackedUp\Temp\WSS_Content.bak'
```

To restore from a **COPY_ONLY** backup, use a command similar to the following:

```SQL
RESTORE DATABASE [WSS_Content_TEST]
FROM DISK = N'E:\NotBackedUp\Temp\WSS_Content.bak'
WITH FILE = 1
    , MOVE N'WSS_Content'
        TO N'E:\Microsoft SQL Server\MSSQL.1\MSSQL\Data\WSS_Content_TEST.MDF'
    , MOVE N'WSS_Content_Log'
        TO N'L:\Microsoft SQL Server\MSSQL.1\MSSQL\Data\WSS_Content_TEST_Log.LDF'
    , NOUNLOAD, STATS = 10
```

Note that when copying a database from one environment to another, you often
need to use the **MOVE** option to specify the new location for the data and log
files (to account for different disk configurations and available disk space).
