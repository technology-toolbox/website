---
title: "Using the Simple Recovery Model for SharePoint Development Environments"
date: 2011-03-19T03:04:00+08:00
excerpt: "A little more than three years ago, I blogged about the default recovery model for various SharePoint databases . In that post, I described how I would often toggle the SQL Server databases in SharePoint development environments from the default Full..."
draft: true
categories: ["My System", "SharePoint"]
tags: ["My System", "MOSS 2007", "SQL Server", "SharePoint 2010"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2011/03/19/using-the-simple-recovery-model-for-sharepoint-development-environments.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/03/19/using-the-simple-recovery-model-for-sharepoint-development-environments.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

A little more than three years ago, I blogged about [the default recovery model for various SharePoint databases](/blog/jjameson/2008/01/18/default-recovery-models-for-sharepoint-databases). In that post, I described how I would often toggle the SQL Server databases in SharePoint development environments from the default Full recovery model to Simple before migrating content.

Since you typically don't care about potential data loss in SharePoint development VMs -- and consequently never bother to configure scheduled database backups -- you might as well *always*use the Simple recovery model for *all*of your development databases. This alleviates the need to periodically backup your transaction logs and also allows you to [use a very small VHD](/blog/jjameson/2011/03/19/creating-small-vhds-lt-1gb-for-hyper-v) for the database log files.

Here's a short SQL script that changes all user databases and the out-of-the-box **model **database to use the Simple recovery model:

```
IF OBJECT_ID('tempdb..#CommandQueue') IS NOT NULL DROP TABLE #CommandQueue

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
    'ALTER DATABASE [' + name + '] SET RECOVERY SIMPLE'
FROM
    sys.databases
WHERE
    name NOT IN ( 'master', 'msdb', 'tempdb' )

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

Note that by changing the **model **database, any new databases created in the development environment (such as content databases created for new Web applications) will be configured to use the Simple recovery model by default.

