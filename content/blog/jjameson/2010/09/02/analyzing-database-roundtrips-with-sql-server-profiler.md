---
title: "Analyzing Database Roundtrips with SQL Server Profiler"
date: 2010-09-02T22:50:00+08:00
excerpt: "One of the \"good habits\" I've developed over the years while creating applications is scrutinizing the interaction between logical or physical tiers. Given the nature of solutions that I'm typically involved with, this often involves examining how many..."
draft: true
categories: ["My System"]
tags: ["My System", "SQL Server"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2010/09/03/analyzing-database-roundtrips-with-sql-server-profiler.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/09/03/analyzing-database-roundtrips-with-sql-server-profiler.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

One of the "good habits" I've developed over the years while creating applications is scrutinizing the interaction between logical or physical tiers. Given the nature of solutions that I'm typically involved with, this often involves examining how many roundtrips are required to the database in order to render a single Web page on a site.

In my experience, poorly performing Web applications are most often due to excessive "chattiness" between the Web server and the database.

My general rule of thumb is the time required for each of the following operations differs by at least one order of magnitude from one to the next:

1. In-memory operations (e.g. finding an object in a Hashtable)
2. Operations that require disk I/O (e.g. reading/writing a file)
3. Fetching data from a database
4. Calling an external Web service

Consequently, if a Web application typically require dozens of roundtrips to the database in order to render a page (and that page is frequently accessed by users), then scalability could certainly be a challenge.

However, trying to identify each database operation  simply by looking at the code is often a futile exercise. This is especially true when building upon frameworks and APIs -- such as object-relational mappers or the SharePoint object model -- particularly when you consider things like "lazy loading."

Instead, I prefer to go straight to the source when identifying the number of database roundtrips. Using SQL Server Profiler, you can quickly -- and reliably -- determine the level of "chattiness" between your application and the backend database.

Note that the **Standard** template selected by default in SQL Server Profiler specifies to trace the following events:

- **Security Audit**
  - **Audit Login**
  - **Audit Logout**
- **Sessions**
  - **ExistingConnection**
- **Stored Procedures**
  - **RPC:Completed**
- **TSQL**
  - **SQL:BatchCompleted**
  - **SQL:BatchStarting**

To identify database roundtrips from an application, I reduce the list of events to the following:

- **Stored Procedures**
  - **RPC:Completed**
- **TSQL**
  - **SQL:BatchCompleted**

I also add the following column filter in order to exclude the "noise" associated with connection pooling:

- **TextData Not Like exec sp\_reset\_connection%**

When analyzing a SharePoint solution, I also add column filters to isolate (as much as possible) the SQL calls initiated by the request for a Web page -- as opposed to, say, those triggered by one of the many SharePoint timer jobs.

Once I've configured the trace, I clear the trace window and then refresh the browser page. I can then see how many database roundtrips are required to render the page simply by looking at the number of rows in SQL Server Profiler (shown in the status bar in the lower right corner). I can then switch between SQL Server Profiler (to clear the trace window again) and the browser to quickly identify which pages require a large number of roundtrips to the database.

When analyzing custom ASP.NET applications, I also like to look at page requests from a "cold start" of the Web application (i.e. after an IIS reset or recycle of the app pool) in order to understand the "warm-up" time for the application when data is fetched from the database and cached for subsequent use.

Provided you have access to the source code for the solution, you can use techniques like batching multiple SELECT statements into a single database roundtrip (or returning multiple result sets from a stored procedure) in order to significantly improve the performance of your application.

Even if you don't have access to the entire source code for your solution (e.g. your solution is built on top of SharePoint or some other API) there are often ways you can optimize in order to reduce database roundtrips. I'll cover a SharePoint-specific example in my next post.

