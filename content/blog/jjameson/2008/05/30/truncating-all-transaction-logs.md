---
title: Truncating All Transaction Logs in SQL Server
date: 2008-05-30T08:32:00-06:00
description:
  One of the SQL scripts that I keep handy in my toolbox is Truncate All
  Transaction Logs.sql . While I would never recommend running this script in a
  Production environment (PROD), I find it to be very helpful for periodically
  freeing up disk space in...
aliases:
  [
    "/blog/jjameson/archive/2008/05/29/truncating-all-transaction-logs.aspx",
    "/blog/jjameson/archive/2008/05/30/truncating-all-transaction-logs.aspx",
  ]
categories: ["My System"]
tags: ["SQL Server", "Toolbox"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2008/05/30/truncating-all-transaction-logs.aspx"
---

One of the SQL scripts that I keep handy in my toolbox is **Truncate All
Transaction Logs.sql**. While I would never recommend running this script in a
Production environment (PROD), I find it to be very helpful for periodically
freeing up disk space in shared Development environments (DEV) and especially on
my local VMs. True, I could instead choose to schedule periodic backups, but
then I'd still have to periodically delete the backup files (or, I suppose, I
could schedule that as well), but, honestly, I really don't care to put that
much effort into managing these environments -- especially since I tend to
periodically "nuke" them from time to time to start fresh.

Here is the script:

```SQL
DROP TABLE #CommandQueue

CREATE TABLE #CommandQueue
(
    ID INT IDENTITY ( 1, 1 )
    , SqlStatement VARCHAR(1000)
)

INSERT INTO #CommandQueue
(
    SqlStatement
)
SELECT
    'BACKUP LOG [' + name + '] WITH TRUNCATE_ONLY'
FROM
    sys.databases
WHERE
    name NOT IN ( 'master', 'model', 'msdb', 'tempdb' )

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

As you can see, there really isn't much to this script. However, what I really
wanted to cover in this post is how I implemented the script, and why I think
this is a good pattern for scripts that perform some operation on an arbitrary
number of objects in a SQL Server database.

Notice that the first thing that I do is create a temporary table that I call
the "command queue." Okay, actually the first thing I do is drop the temporary
table if it exists, just in case I need to run the script multiple times in a
single session (not necessarily all that helpful when truncating transaction
logs, but remember this is more about the pattern than this particular script).

While I could certainly choose to bypass the "command queue" and simply execute
the SQL statements directly, I find this approach to be much more robust in
terms of error handling. If something unexpected happens during the execution of
the script, I can easily resume processing after recovering from the error
(without having to completely start over again).

The rest of the script really requires no explanation. You can see that once
I've "queued" up all of the commands to be run, I simply process them one at a
time in a FIFO (First-In-First-Out) manner -- nothing special there.

In my [next post](/blog/jjameson/2008/05/30/shrinking-all-database-files), I
share another useful script that follows the same pattern and potentially frees
up even more disk space than simply truncating the transaction logs.
