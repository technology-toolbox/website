---
title: "\"Incorrect syntax near 'COLLATE'.\" Error When Trying to Join a SharePoint Farm"
date: 2007-08-21T07:32:00-06:00
excerpt: "I encountered another nasty bug this morning while rebuilding our Microsoft Office SharePoint Server (MOSS) 2007 Development environment (DEV). Since the time I originally created DEV, I installed SQL Server SP2 and also restored several legacy databases..."
aliases: ["/blog/jjameson/archive/2007/08/20/incorrect-syntax-near-collate-when-trying-to-join-a-sharepoint-farm.aspx", "/blog/jjameson/archive/2007/08/21/incorrect-syntax-near-collate-when-trying-to-join-a-sharepoint-farm.aspx"]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2007/08/21/incorrect-syntax-near-collate-when-trying-to-join-a-sharepoint-farm.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/08/21/incorrect-syntax-near-collate-when-trying-to-join-a-sharepoint-farm.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

I encountered another nasty bug this morning while rebuilding our Microsoft
Office SharePoint Server (MOSS) 2007 Development environment (DEV). Since the
time I originally created DEV, I installed SQL Server SP2 and also restored
several legacy databases to support the development of the next version ("v2")
of our solution.

After running the SharePoint Products and Technologies Configuration Wizard on
the SSP server to create the farm, I ran the wizard on the first front-end Web
server in order to join it to the new farm. On the second step of the
configuration wizard, I left the default option selected (**Yes, I want to
connect to an existing server farm**) and then clicked **Next**. On the
**Specify Configuration Database Settings** step, I typed the name of the SQL
Server and then clicked **Retrieve Database Names**.

At this point, I was presented with the oh-so-lovely "Unhandled exception"
dialog displayed by the .NET Framework when something very bad happens.

**Error message:**

{{< blockquote "font-italic text-danger" >}}

Unhandled exception has occurred in your application. If you click Continue, the
application will ignore this error and attempt to continue. If you click Quit,
the application will close immediately.

Incorrect syntax near 'COLLATE'. You may need to set the compatibility level of
the current database to a higher value to enable this feature. See help for the
stored procedure sp\_dbcmptlevel.

{{< /blockquote >}}

**Details:**

```
See the end of this message for details on invoking just-in-time (JIT) debugging instead of this dialog box.

************** Exception Text **************
System.Data.SqlClient.SqlException: Incorrect syntax near 'COLLATE'. You may need to
set the compatibility level of the current database to a higher value to enable this
feature. See help for the stored procedure sp_dbcmptlevel.
at System.Data.SqlClient.SqlConnection.OnError(SqlException exception, Boolean breakConnection)
at System.Data.SqlClient.SqlInternalConnection.OnError(SqlException exception, Boolean breakConnection)
at System.Data.SqlClient.TdsParser.ThrowExceptionAndWarning(TdsParserStateObject stateObj)
at System.Data.SqlClient.TdsParser.Run(RunBehavior runBehavior, SqlCommand cmdHandler, SqlDataReader dataStream, BulkCopySimpleResultSet bulkCopyHandler, TdsParserStateObject stateObj)
at System.Data.SqlClient.SqlCommand.FinishExecuteReader(SqlDataReader ds, RunBehavior runBehavior, String resetOptionsString)
at System.Data.SqlClient.SqlCommand.RunExecuteReaderTds(CommandBehavior cmdBehavior, RunBehavior runBehavior, Boolean returnStream, Boolean async)
at System.Data.SqlClient.SqlCommand.RunExecuteReader(CommandBehavior cmdBehavior, RunBehavior runBehavior, Boolean returnStream, String method, DbAsyncResult result)
at System.Data.SqlClient.SqlCommand.InternalExecuteNonQuery(DbAsyncResult result, String methodName, Boolean sendToPipe)
at System.Data.SqlClient.SqlCommand.ExecuteNonQuery()
at Microsoft.SharePoint.PostSetupConfiguration.SqlSession.ExecuteNonQuery(SqlCommand command)
at Microsoft.SharePoint.PostSetupConfiguration.SqlServerHelper.DatabaseTableWithColumnExists(String table, String column)
at Microsoft.SharePoint.PostSetupConfiguration.SqlServerHelper.GetV3WSSConfigurationDatabases()
at Microsoft.SharePoint.PostSetupConfiguration.ConnectConfigurationDbForm.GetDatabasesButtonClickEventHandler(Object sender, EventArgs e)
at System.Windows.Forms.Control.OnClick(EventArgs e)
at System.Windows.Forms.Button.OnClick(EventArgs e)
at System.Windows.Forms.Button.WndProc(Message& m)
at System.Windows.Forms.Control.ControlNativeWindow.OnMessage(Message& m)
at System.Windows.Forms.Control.ControlNativeWindow.WndProc(Message& m)
at System.Windows.Forms.NativeWindow.Callback(IntPtr hWnd, Int32 msg, IntPtr wparam, IntPtr lparam)
```

I performed a quick search on our internal KB (which includes support incidents)
and discovered that there is a known issue when the SQL Server has a database
where the compatibility is set to 70. As I mentioned at the beginning of this
post, I had recently restored some legacy databases in order to support the
migration from the legacy systems to the new MOSS-based solution.

The solution documented in the SR (Service Request) was to backup the database
(where the compatibility level is set to 70), change the compatibility level,
and then proceed to re-run the SharePoint Products and Technologies
Configuration Wizard. Instead, I chose to simply type in the name of the
configuration database (we are using the default, **SharePoint\_Config**) as
well as the service account name and password. The configuration then completed
without error.
