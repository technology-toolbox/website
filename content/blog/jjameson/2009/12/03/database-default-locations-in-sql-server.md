---
title: "Database Default Locations in SQL Server"
date: 2009-12-03T05:30:00-07:00
excerpt: "I've mentioned before the importance of using multiple \"spindles\" when working with large SQL Server databases. 
 Generally speaking, the recommendation is to use different RAID 1+0 arrays for data and log files -- and depending on the size and load..."
aliases: ["/blog/jjameson/archive/2009/12/02/database-default-locations-in-sql-server.aspx", "/blog/jjameson/archive/2009/12/03/database-default-locations-in-sql-server.aspx"]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "SQL Server"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/12/03/database-default-locations-in-sql-server.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/12/03/database-default-locations-in-sql-server.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

I've mentioned before the importance of using multiple "spindles" when working
with large SQL Server databases.

Generally speaking, the recommendation is to use different RAID 1+0 arrays for
data and log files -- and depending on the size and load of your database, you
may also need to isolate data files on individual RAID 1+0 arrays.

For really large OLTP databases that require lots of
[IOPS](http://en.wikipedia.org/wiki/IOPS), additional arrays might also be
required to split the log files (although, honestly, I haven't worked on any
projects where this has been necessary).

One of the configuration steps that I always recommend for any Production SQL
Server environment is to configure the default database locations to ensure your
"data I/O" is isolated from your "log I/O" (both of which should be isolated
from other I/O -- specifically, the paging file).

For example, suppose you have the following disk configuration on your SQL
Server cluster:

- C: (Local Disk) - RAID 1 array for operating system files, program files,
  and paging file
- D: (DATA01) - RAID 1+0 array for data files
- L: (LOG01) - RAID 1+0 array for log files
- Q: (Quorum)

In this simplistic example, the D:, L:, and Q: drives are LUNs on the backend
SAN, whereas the C: drive is DAS (Direct Attached Storage). Hopefully, the
Storage team responsible for managing your SAN provides dedicated physical
drives for D: and L: -- although, honestly, I've seen several enterprise
organizations that don't dedicate these drives and subsequently experience
performance issues later on.

Given the above disk configuration, SQL Server Management Studio should be used
to configure the default database locations similar to the following:

- Server Properties
  - Database Settings
    - Default database locations
      - Data: D:\NotBackedUp\Microsoft SQL
        Server\MSSQL10.MSSQLSERVER\MSSQL\DATA
      - Log: L:\NotBackedUp\Microsoft SQL
        Server\MSSQL10.MSSQLSERVER\MSSQL\DATA

You certainly don't have to use the
[NotBackedUp](/blog/jjameson/2007/03/22/backedup-and-notbackedup) folder if you
don't want to (that's just the standard that I've been using for years).

Once you've configured the default database locations, anytime you create a new
database -- for example, when you create a new Web application or content
database in Microsoft Office SharePoint Server (MOSS) 2007 -- the data and log
files will be placed on the desired drives (i.e. D: and L:, respectively).

Of course, when creating MOSS 2007 databases for a Production (or Test)
environment, you still need to resize them appropriately in order to avoid
having them simply auto-grow from a very small initial size (in order to avoid
fragmentation and optimize performance).

Suppose you need to determine the default database locations through SQL
(perhaps because you are scripting the process to create your databases). If you
fire up SQL Server Profiler and start a new trace (the default trace options are
fine), you will find that SQL Server Management Studio executes the following
SQL statements when viewing the **Database Settings** page in the **Server
Properties** window:

```
declare @RegPathParams sysname
declare @Arg sysname
declare @Param sysname
declare @MasterPath nvarchar(512)
declare @LogPath nvarchar(512)
declare @ErrorLogPath nvarchar(512)
declare @n int

select @n=0
select @RegPathParams=N'Software\Microsoft\MSSQLServer\MSSQLServer'+'\Parameters'
select @Param='dummy'
while(not @Param is null)
begin
    select @Param=null
    select @Arg='SqlArg'+convert(nvarchar,@n)

    exec master.dbo.xp_instance_regread
        N'HKEY_LOCAL_MACHINE',
        @RegPathParams,
        @Arg,
        @Param OUTPUT

    if(@Param like '-d%')
    begin
        select @Param=substring(@Param, 3, 255)
        select @MasterPath=substring(
            @Param,
            1,
            len(@Param) - charindex('\', reverse(@Param)))
    end
    else if(@Param like '-l%')
    begin
        select @Param=substring(@Param, 3, 255)
        select @LogPath=substring(
            @Param,
            1,
            len(@Param) - charindex('\', reverse(@Param)))
    end
    else if(@Param like '-e%')
    begin
        select @Param=substring(@Param, 3, 255)
        select @ErrorLogPath=substring(
            @Param,
            1,
            len(@Param) - charindex('\', reverse(@Param)))
    end

    select @n=@n+1
end

print 'LogPath = ' + @LogPath
print 'MasterPath = ' + @MasterPath
```

Note that I truncated the actual SQL batch and added the `print` statements.

