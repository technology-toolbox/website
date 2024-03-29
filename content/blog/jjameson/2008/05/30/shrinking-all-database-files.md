---
title: Shrinking All Database Files in SQL Server
date: 2008-05-30T08:50:00-06:00
description:
  "Here is another SQL script that I keep handy in my toolbox: Shrink All
  Database Files.sql . Unlike the script that I shared in my previous post that
  simply truncated all transaction logs to free up disk space, this script is
  suitable for running in a..."
aliases:
  [
    "/blog/jjameson/archive/2008/05/29/shrinking-all-database-files.aspx",
    "/blog/jjameson/archive/2008/05/30/shrinking-all-database-files.aspx",
  ]
categories: ["SharePoint", "My System"]
tags: ["MOSS 2007", "SQL Server", "Toolbox"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2008/05/30/shrinking-all-database-files.aspx"
---

Here is another SQL script that I keep handy in my toolbox: **Shrink All
Database Files.sql**. Unlike the script that I shared in my
[previous post](/blog/jjameson/2008/05/30/truncating-all-transaction-logs) that
simply truncated all transaction logs to free up disk space, this script is
suitable for running in a Production environment (PROD), as well as in
non-production environments, such as a shared Development environment (DEV).

Here is the script:

```SQL
DROP TABLE #CommandQueue

CREATE TABLE #CommandQueue
(
    ID INT IDENTITY ( 1, 1 )
    , SqlStatement VARCHAR(1000)
)

INSERT INTO    #CommandQueue
(
    SqlStatement
)
SELECT
    'USE [' + A.name + '] DBCC SHRINKFILE (N''' + B.name + ''' , 1)'
FROM
    sys.databases A
    INNER JOIN sys.master_files B
    ON A.database_id = B.database_id
WHERE
    A.name NOT IN ( 'master', 'model', 'msdb', 'tempdb' )

DECLARE @id INT

SELECT @id = MIN(ID)
FROM #CommandQueue

WHILE @id IS NOT NULL
BEGIN
    DECLARE @sqlStatement VARCHAR(1000)

    SELECT
        @sqlStatement = SqlStatement
    FROM
        #CommandQueue
    WHERE
        ID = @id

    PRINT 'Executing ''' + @sqlStatement + '''...'

    EXEC (@sqlStatement)

    DELETE FROM #CommandQueue
    WHERE ID = @id

    SELECT @id = MIN(ID)
    FROM #CommandQueue
END
```

As you can see, this script follows the same pattern that I described in my
previous post.

I have found this script to be especially useful when working with Microsoft
Office SharePoint Server (MOSS) 2007, because I sometimes migrate large amounts
of content when working on certain features (particularly Search) but later
decide to remove the content and need to recover the disk space on my VM.
