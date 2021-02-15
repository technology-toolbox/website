---
title: "Refresh Development and Test environments with Production database (a.k.a. Building TechnologyToolbox.com, part 7)"
date: 2011-11-14T07:53:58-07:00
excerpt: "In this post, I'll show you how I quickly restore the Production database for TechnologyToolbox.com to the corresponding Development and Test environments..."
draft: true
categories: ["Development", "My System"]
tags: ["My System", "Subtext", "SQL Server"]
---

In this post, I'll show you how I quickly restore the Production database
for TechnologyToolbox.com to the corresponding Development and Test environments.

### Introduction

In
[my previous post](/blog/jjameson/2011/11/13/building-technologytoolbox-com-part-6), I explained how I migrated blog posts from
[my old MSDN blog](http://blogs.msdn.com/b/jjameson/) to Subtext.
While developing the migration utility, I ran the migration process countless
times on one of my development VMs (or what I typically refer to as "LOCAL").
Then, once the migration tool was nearing completion, I began running it in
the Development integration environment (DEV). For final validation purposes,
I ran the migration tool in the Test environment (TEST) a few times while I
worked out the remaining bugs and issues. Eventually I performed the migration
process one last time in the Production environment (PROD).

From that point on, the multiple environments gradually became more and more
out-of-sync. For example, adding new blog posts or editing existing blog posts
in PROD meant the corresponding changes would not be reflected in DEV and TEST.

One of the best practices I recommend to customers is to make their Test
environments look as much as possible like their Production environments. One
aspect of this recommendation is to periodically synchronize the TEST database(s)
with the data from PROD. The easiest way to achieve this is by restoring PROD
database backups.

However, there is usually a little more to it than simply restoring a backup.
For example, you typically need to reconfigure security to account for differences
between the environments. You also may need to "scrub" the data to remove sensitive
information, update passwords, etc.

For the remainder of this post, it is important to first review the infrastructure
for the various environments used for TechnologyToolbox.com.

The Production environment for TechnologyToolbox.com is currently hosted
by [WinHost](http://www.winhost.com), as shown in the following figure.
Separate servers are used for database services (i.e. SQL Server) and the Web
tier.

![Infrastructure](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/8/r_Technology-Toolbox-Infrastructure.jpg)

    	Figure 1: Infrastructure

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/8/o_Technology-Toolbox-Infrastructure.jpg)

While I generally prefer to use Windows Authentication with SQL Server --
for reasons I described in
[a post from way back in 2007](/blog/jjameson/2007/03/23/sql-server-authentication-modes) -- this is neither supported (nor desired)
in the hosted scenario. If the reasons for this are not immediately clear, I'll
explain why in the following sections.

### Refreshing the TEST database for TechnologyToolbox.com

In the Test environment, the Web application runs on one server (CYCLOPS)
whereas the database runs on a different server (BEAST). In this regard, it
is very similar to the Production environment (i.e. distributed physical tiers).
However, unlike PROD, the Web tier in TEST does not host numerous Web applications
for different companies.

In the WinHost/Production environment, each Web application gets its own
application pool (for stability and security purposes) but each app pool does
not run as a unique service account (presumably to make it easier for the hosting
provider to setup and maintain). Consequently while it might be possible to
use Windows Authentication between the Web tier and the data tier (assuming
the two servers are in the same domain or separate domains with a trust relationship),
this would expose a huge security hole because it would allow, for example,
other Web applications running on the same Web server to access the Technology
Toolbox database. Similarly, it would allow the Technology Toolbox Web application
to access the databases for other companies. To avoid this vulnerability, SQL
Server Authentication is the only option available in the Production environment.

In TEST, however, *Windows Authentication* is used between the Web
tier and the data tier. In order to allow the Web application to communicate
with the remote SQL Server database, the app pool in TEST is configured to run
as **NetworkService**. In addition, the machine account for the
Web server (i.e. CYCLOPS) must be granted access to the database. This is accomplished
by adding **TECHTOOLBOX\CYCLOPS$** to the **db\_datareader** and **db\_datawriter** database roles.

Here is the SQL script I wrote to refresh the Test environment from Production.
Also note that I update the **subtext\_Config** table to reflect
the host header used in TEST (i.e.
[http://www-test.technologytoolbox.com](http://www-test.technologytoolbox.com)).

#### Restore PROD to TEST.sql

```
USE tempdb
GO
DROP DATABASE Caelum
GO

RESTORE DATABASE [Caelum]
    FROM  DISK = N'C:\NotBackedUp\Backups\Caelum\Caelum_backup_2011-11-14-14-35.bak'
    WITH  FILE = 1
        ,  MOVE N'Caelum_data' TO N'C:\NotBackedUp\Microsoft SQL Server\MSSQL10_50.MSSQLSERVER\MSSQL\DATA\Caelum.mdf'
        ,  MOVE N'Caelum_log' TO N'C:\NotBackedUp\Microsoft SQL Server\MSSQL10_50.MSSQLSERVER\MSSQL\Data\Caelum.ldf'
        ,  NOUNLOAD
        ,  STATS = 10
GO

IF NOT EXISTS (
    SELECT * FROM master..syslogins
    WHERE name = 'TECHTOOLBOX\CYCLOPS$')
BEGIN
    CREATE LOGIN [TECHTOOLBOX\CYCLOPS$]
    FROM WINDOWS WITH DEFAULT_DATABASE=[tempdb]
END

USE [Caelum]
GO
CREATE USER [TECHTOOLBOX\CYCLOPS$] FOR LOGIN [TECHTOOLBOX\CYCLOPS$]
GO
EXEC sp_addrolemember N'db_datareader', N'TECHTOOLBOX\CYCLOPS$'
GO
EXEC sp_addrolemember N'db_datawriter', N'TECHTOOLBOX\CYCLOPS$'
GO

UPDATE subtext_Config
SET Host = N'www-test.technologytoolbox.com'
WHERE Title = 'Random Musings of Jeremy Jameson'
```

### Refreshing the DEV database for TechnologyToolbox.com

In the Development integration environment, the Web application and database
run on the same server (CYCLOPS-DEV). Since there is no need to authenticate
between servers, the application pool in DEV is configured to use the default
(i.e. **ApplicationPoolIdentity** -- which, in this case, translates
to **IIS APPPOOL\www-dev.technologytoolbox.com**).

Consequently, rather than adding the machine account to the database roles,
the script adds **IIS APPPOOL\www-dev.technologytoolbox.com**.
The **subtext\_Config** table is also updated to reflect the host
header used in DEV (i.e. [http://www-dev.technologytoolbox.com](http://www-dev.technologytoolbox.com)).

#### Restore PROD to DEV.sql

```
USE tempdb
GO
DROP DATABASE Caelum
GO

RESTORE DATABASE [Caelum]
    FROM  DISK = N'\\beast\Backups\Caelum\Caelum_backup_2011-11-14-14-35.bak'
    WITH  FILE = 1
        ,  MOVE N'Caelum_data' TO N'D:\NotBackedUp\Microsoft SQL Server\MSSQL10_50.MSSQLSERVER\MSSQL\DATA\Caelum.mdf'
        ,  MOVE N'Caelum_log' TO N'L:\NotBackedUp\Microsoft SQL Server\MSSQL10_50.MSSQLSERVER\MSSQL\Data\Caelum.ldf'
        ,  NOUNLOAD
        ,  STATS = 10
GO

IF NOT EXISTS (
    SELECT * FROM master..syslogins
    WHERE name = 'IIS APPPOOL\www-dev.technologytoolbox.com')
BEGIN
    CREATE LOGIN [IIS APPPOOL\www-dev.technologytoolbox.com]
    FROM WINDOWS WITH DEFAULT_DATABASE=[tempdb]
END

USE [Caelum]
GO
CREATE USER [IIS APPPOOL\www-dev.technologytoolbox.com]
FOR LOGIN [IIS APPPOOL\www-dev.technologytoolbox.com]
GO
EXEC sp_addrolemember
    N'db_datareader',
    N'IIS APPPOOL\www-dev.technologytoolbox.com'
GO
EXEC sp_addrolemember
    N'db_datawriter',
    N'IIS APPPOOL\www-dev.technologytoolbox.com'
GO

UPDATE subtext_Config
SET Host = N'www-dev.technologytoolbox.com'
WHERE Title = 'Random Musings of Jeremy Jameson'
```

