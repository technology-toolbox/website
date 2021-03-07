---
title: "Backing Up User Databases in SQL Server (and SQL Server Express)"
date: 2008-05-30T09:22:00-06:00
excerpt: "Since I appear to be on a roll with my blog this morning, I figured that I should write one more post about SQL Server before I get back to my \"day job.\" 
 I typically use SQL Server Management Studio to configure and schedule database backups, because..."
aliases: ["/blog/jjameson/archive/2008/05/29/backing-up-user-databases-in-sql-server-and-sql-server-express.aspx", "/blog/jjameson/archive/2008/05/30/backing-up-user-databases-in-sql-server-and-sql-server-express.aspx"]
draft: true
categories: ["Infrastructure", "My System"]
tags: ["SQL Server", "WSUS", "Toolbox"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2008/05/30/backing-up-user-databases-in-sql-server-and-sql-server-express.aspx](http://blogs.msdn.com/b/jjameson/archive/2008/05/30/backing-up-user-databases-in-sql-server-and-sql-server-express.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Since I appear to be on a roll with my blog this morning, I figured that I
should write one more post about SQL Server before I get back to my "day job."

I typically use SQL Server Management Studio to configure and schedule database
backups, because the **Maintenance Plan Wizard** makes it very quick and easy to
click through a few screens and select the appropriate options. However, on some
of the servers running in the
["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) (a.k.a.
my basement), I only have SQL Server Express installed, not the full SQL Server
product. For example, on my server that runs Windows Server Update Services
(WSUS), I use SQL Server Express as the "backend" storage solution, because for
this particular scenario, I want to keep the database local instead of relying
on a separate server. However, I still want to ensure that I have periodic
backups of the databases.

In order to make this as painless as possible, I wrote the following script:

```
USE [Tools]
GO
/****** Object:  StoredProcedure [dbo].[BackupUserDatabases]    Script Date: 03/15/2007 07:55:44 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE PROC [dbo].[BackupUserDatabases]
(
    @backupType VARCHAR(15)
)
AS

DECLARE @backupFolder VARCHAR(255)

SET @backupFolder = 'C:\BackedUp\Microsoft SQL Server\MSSQL.1\MSSQL\Backup\' + @backupType

DECLARE @timestamp DATETIME
DECLARE @dateString VARCHAR(8)
DECLARE @timeString VARCHAR(12)

SELECT @timestamp = GETDATE()

SELECT @dateString = CONVERT(VARCHAR(8), @timestamp, 112)

SELECT @timeString = CONVERT(VARCHAR(12), @timestamp, 14)

-- Remove seconds from timestamp
SET @timeString = LEFT(REPLACE(@timeString,':',''), 4)

DECLARE @databases TABLE
(
    ID INT IDENTITY ( 1, 1 )
    , DatabaseName SYSNAME
)

INSERT INTO
    @databases
    SELECT name
    FROM master.dbo.sysdatabases
    WHERE name NOT IN ( 'master', 'model', 'msdb', 'tempdb' )

DECLARE @id TINYINT
SELECT @id = MIN ( ID ) FROM @databases

WHILE @id IS NOT NULL BEGIN
    DECLARE @databaseName SYSNAME
    SELECT @databaseName = DatabaseName FROM @databases WHERE ID = @id

    DECLARE @backupFileName VARCHAR(512)
    SELECT @backupFileName = @databaseName + '_backup_' +@dateString + @timeString + '.bak'

    IF @backupType = 'Full'
    BEGIN
        EXEC ('BACKUP DATABASE [' + @databaseName + '] TO DISK =''' + @backupFolder + '\' + @BackupFileName + '''')
    END
    ELSE IF @backupType = 'Differential'
    BEGIN
        EXEC ('BACKUP DATABASE [' + @databaseName + '] TO DISK =''' + @backupFolder + '\' + @BackupFileName + ''' WITH DIFFERENTIAL')
    END
    ELSE IF @backupType = 'Transaction Log'
    BEGIN
        DECLARE @recoveryModel SQL_VARIANT

        SELECT @recoveryModel = DATABASEPROPERTYEX(@databaseName, 'Recovery')

        IF @recoveryModel <> 'SIMPLE'
        BEGIN
            EXEC ('BACKUP LOG [' + @databaseName + '] TO DISK =''' + @backupFolder + '\' + @BackupFileName + '''')
        END
    END
    ELSE
    BEGIN
        RAISERROR ('Invalid backup type', 16, 1)
    END

    DELETE FROM @databases WHERE ID = @id

    SELECT @id = MIN ( ID ) FROM @databases
END
```

Well, technically, it's a stored procedure, but nevertheless I still keep the
script to generate the sproc in my toolbox: **BackupUserDatabases.sql**. Notice
that I create the sproc in a separate database (I arbitrarily chose the name
**Tools**).

I can then schedule full, differential, and transaction log backups using
scheduled tasks, as shown below.

{{< figure src="https://assets.technologytoolbox.com/blog/jjameson/Images/My-System/Scheduled-Tasks-COLOSSUS-600x119.jpg" alt="Scheduled tasks for backing up databases" height="119" width="600" title="Figure 1: Scheduled tasks for backing up databases" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/My-System/Scheduled-Tasks-COLOSSUS-1131x224.jpg)

Here is the command behind one of the scheduled tasks (you can easily deduce the
others):

```
"C:\Program Files\Microsoft SQL Server\90\Tools\Binn\SQLCMD.EXE" -S .\SQLExpress -d Tools -Q "EXEC BackupUserDatabases @backupType='Full'"
```

Lastly, note that I have a separate server periodically ROBOCOPY the backup
files off of this server to another location -- just in case the WSUS server
happens to catch on fire or some other act of God completely wipes out the local
database backups ;-)

