---
title: "Counting Rows in All Database Tables in SQL Server"
date: 2008-05-30T02:56:00-07:00
excerpt: "Here is yet another of the SQL scripts that I like to keep handy in my toolbox: Count Records in All Tables.sql 
 Sometimes when I get \"dropped into\" a consulting situation with a new customer, I need to quickly get acquainted with one or more of their..."
draft: true
categories: ["My System"]
tags: ["SQL Server", "Toolbox"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2008/05/30/counting-rows-in-all-database-tables.aspx](http://blogs.msdn.com/b/jjameson/archive/2008/05/30/counting-rows-in-all-database-tables.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

Here is yet another of the SQL scripts that I like to keep handy in my toolbox: **Count Records in All Tables.sql**

Sometimes when I get "dropped into" a consulting situation with a new customer, I need to quickly get acquainted with one or more of their SQL Server databases. One of the first things I usually like to know is: "What are the largest tables in the database in terms of the number of rows?"

While you could certainly craft some SQL to `SELECT COUNT(*)` from each user table, this is very inefficient. A much better way is to simply query the system tables as shown below:

```
SELECT
    sysobjects.Name
    , sysindexes.Rows
FROM
    sysobjects
    INNER JOIN sysindexes
    ON sysobjects.id = sysindexes.id
WHERE
    type = 'U'
    AND sysindexes.IndId < 2
ORDER BY
    sysobjects.Namecode
```
