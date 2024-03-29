---
title: Bug in MOSS 2007 FBA with Insufficient Permissions in SQL Server
date: 2009-10-12T05:58:00-06:00
description:
  "A couple of weeks ago I was setting up Forms-Based Authentication (FBA) on my
  new development VM for Microsoft Office SharePoint Server (MOSS) 2007, and I
  spent a few hours troubleshooting why I couldn't add a custom role
  (\"Authenticated Users\") to a..."
aliases:
  [
    "/blog/jjameson/archive/2009/10/11/bug-in-moss-2007-fba-with-insufficient-permissions-in-sql-server.aspx",
    "/blog/jjameson/archive/2009/10/12/bug-in-moss-2007-fba-with-insufficient-permissions-in-sql-server.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/10/12/bug-in-moss-2007-fba-with-insufficient-permissions-in-sql-server.aspx"
---

A couple of weeks ago I was setting up Forms-Based Authentication (FBA) on my
new development VM for Microsoft Office SharePoint Server (MOSS) 2007, and I
spent a few hours troubleshooting why I couldn't add a custom role
("Authenticated Users") to a SharePoint site.

Just like I'd done a couple of times before, I meticulously followed Steve
Peschka's article on MSDN --
[Forms Authentication in SharePoint Products and Technologies (Part 1)](http://msdn.microsoft.com/en-us/library/bb975136.aspx)
-- that provides lots of details with regards to ensuring you have all the right
Web.config entries (connection strings, membership element, etc.) necessary to
make FBA work.

I'd also granted the following permissions in SQL Server to the service account
for the application pool corresponding to my Web application:

- **aspnet_Membership_BasicAccess**
- **aspnet_Roles_BasicAccess**

Like I said before, I had previously configured FBA in MOSS 2007 for a couple of
previous projects, but this time, in addition to actually authenticating users
(i.e. "membership"), I also needed to restrict access to certain areas of the
site based on groups of FBA users (i.e. "roles").

However, when I tried adding my custom **Authenticated Users** role to a
SharePoint group, the only thing SharePoint would recognize was **NT
AUTHORITY\authenticated users**. I even tried typing in the fully qualified role
name (**FabrikamSqlRoleProvider:Authenticated Users**), but SharePoint didn't
recognize that either.

The really frustrating part is that I could add users from my ASP.NET membership
database to SharePoint groups, I just couldn't add roles. This certainly made me
think that I had the necessary Web.config entries.

I then fired up SQL Server Profiler to verify that SharePoint was at least
_trying_ to query my ASP.NET database when adding a role to a SharePoint group.
Sure enough, I saw the following statement in the profiler trace:

```SQL
exec dbo.aspnet_Roles_RoleExists
    @ApplicationName=N'Fabrikam Portal',
    @RoleName=N'Authenticated Users'
```

Hmmm...from this I could tell that SharePoint was definitely attempting to
validate the custom role. I then attached the debugger and set it to break on
all exceptions (the approach I frequently use when I am completely baffled by
something).

That's when I found the problem, namely a `SqlException`:

{{< div-block "errorMessage" >}}

> The EXECUTE permission was denied on the object 'aspnet_Roles_RoleExists',
> database 'FabrikamPortal', schema 'dbo'.

{{< /div-block >}}

Ugh...it turns out the **aspnet_Roles_RoleExists** stored procedure is by
default only granted EXECUTE permission to the **aspnet_Roles_ReportingAccess**
database role within SQL Server. Unfortunately, SharePoint was simply
"swallowing" that `SqlException` and assuming the role simply did not exist. I
don't know about you, but I consider a "swallowed exception" like this to be a
bug. Others may disagree, but that's my opinion.

The lesson learned here is that when using Forms-Based Authentication and the
out-of-the-box ASP.NET membership and role providers, your service account needs
to be added to the following database roles in your ASP.NET database:

- **aspnet_Membership_BasicAccess**
- **aspnet_Membership_ReportingAccess**
- **aspnet_Roles_BasicAccess**
- **aspnet_Roles_ReportingAccess**

The reason you should add it the **aspnet_Membership_ReportingAccess** database
role -- in addition to the **aspnet_Roles_ReportingAccess** database role -- is
that the sprocs that allow you to do partial matching on user names (e.g.
**aspnet_Membership_FindUsersByName**) are by default only granted EXECUTE
permission to the **aspnet_Membership_ReportingAccess** database role (not
**aspnet_Membership_BasicAccess**).

In other words, when I said earlier that I could add FBA users to a SharePoint
group when my service account was a member of the
**aspnet_Membership_BasicAccess** database role, that only worked because I
typed in the full username -- which gets validated using the
**aspnet_Membership_GetUserByName** sproc (which _is_ granted EXECUTE permission
to the **aspnet_Membership_BasicAccess** database role).

By the way, here's some script to automatically create a user in SQL Server for
the service account and add it to the necessary database roles:

```SQL
USE [FabrikamPortal]
GO

CREATE USER [TECHTOOLBOX\svc-web-fabrikam-dev]
FOR LOGIN [TECHTOOLBOX\svc-web-fabrikam-dev]
WITH DEFAULT_SCHEMA=[dbo]
GO

EXEC sp_addrolemember N'aspnet_Membership_BasicAccess', N'TECHTOOLBOX\svc-web-fabrikam-dev'
GO
EXEC sp_addrolemember N'aspnet_Membership_ReportingAccess', N'TECHTOOLBOX\svc-web-fabrikam-dev'
GO
EXEC sp_addrolemember N'aspnet_Roles_BasicAccess', N'TECHTOOLBOX\svc-web-fabrikam-dev'
GO
EXEC sp_addrolemember N'aspnet_Roles_ReportingAccess', N'TECHTOOLBOX\svc-web-fabrikam-dev'
GO
```

You'll obviously need to replace the name of your ASP.NET database and service
account accordingly.

Isn't it fun conforming to the principle of least privilege?!
