---
title: "\"The workbook cannot be opened\" Error with SharePoint Server 2010 (and TFS 2010)"
date: 2010-05-04T08:50:00-06:00
excerpt:
  In an earlier post today, I described how I recently upgraded from Team
  Foundation Server 2008 (and Windows SharePoint Services v3) to TFS 2010 (and
  SharePoint Server 2010). While most of the upgrade went fairly smooth, during
  the process I discovered...
aliases:
  [
    "/blog/jjameson/archive/2010/05/03/the-workbook-cannot-be-opened-error-with-sharepoint-server-2010-and-tfs-2010.aspx",
    "/blog/jjameson/archive/2010/05/04/the-workbook-cannot-be-opened-error-with-sharepoint-server-2010-and-tfs-2010.aspx",
  ]
draft: true
categories: ["Development", "SharePoint"]
tags: ["TFS", "SharePoint 2010"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/05/04/the-workbook-cannot-be-opened-error-with-sharepoint-server-2010-and-tfs-2010.aspx"
---

In an
[earlier post](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010-overview)
today, I described how I recently upgraded from Team Foundation Server 2008 (and
Windows SharePoint Services v3) to TFS 2010 (and SharePoint Server 2010).

While most of the upgrade went fairly smooth, during the process I discovered
(what I believe to be) a bug in SharePoint Server 2010 with Excel Services.

Note that if you use Microsoft Office SharePoint Server (MOSS) 2007 or
SharePoint Server 2010 when creating new TFS project sites, you get some really
rich dashboard functionality that is enabled through Excel Services (which, as I
noted earlier, is a compelling reason to upgrade from WSS to SharePoint Server
2010 for TFS project sites).

Also note that Jon Tsao wrote a blog post a couple of months ago with the steps
to
[configure SharePoint Server 2010 for dashboard compatibility with TFS 2010](http://blogs.msdn.com/team_foundation/archive/2010/03/06/configuring-sharepoint-server-2010-beta-for-dashboard-compatibility-with-tfs-2010-beta2-rc.aspx).
[Be aware that Jon's post was based on a pre-release version of SharePoint
Server 2010, so a couple of the screenshots and corresponding configuration
steps are out-of-date (for example, the link you click to configure Excel
Services is no longer accessible via the **Manage services on server** link
under the **System Settings** section of SharePoint Central Administration.
Instead, you need to click the **Manage service applications** link under the
**Application Management** section.]

I followed Jon's post to configure my environment, but I found that when I
browsed to a new TFS project site, all of the Excel Web Parts displayed the
following error:

{{< blockquote "font-italic text-danger" >}}

The workbook cannot be opened.

{{< /blockquote >}}

Looking at the event log on the SharePoint/TFS server, I found the following
error occurred each time I requested the dashboard page:

```Text
Source: Microsoft-SharePoint Products-SharePoint Foundation
Event ID: 3760
Task Category: Database
Level: Critical
User: TECHTOOLBOX\svc-spserviceapp-tst
Computer: cyclops-test.corp.technologytoolbox.com
Description:
SQL Database 'WSS_Content' on SQL Server instance 'beast-test' not found. Additional error information from SQL Server is included below.

Cannot open database "WSS_Content" requested by the login. The login failed.
Login failed for user 'TECHTOOLBOX\svc-spserviceapp-tst'.
```

Note that in my environment, I use different service accounts for the TFS Web
application and SharePoint service applications (e.g. Excel Services and the
Secure Store Service). I *believe* Jon configured his environment such that the
Web application and SharePoint service applications use the same service
account.

I say this because looking at SQL Server Management Studio, I discovered that
while the service account for the app pool of the TFS Web application
(TECHTOOLBOX\svc-web-tfs-test) had access to the underlying SharePoint content
database (WSS\_Content), the service account for Excel Services
(TECHTOOLBOX\svc-spserviceapp-tst) did not.

This explained the error that I was seeing on my test SharePoint/TFS server
(CYCLOPS-TEST) -- which seems like a bug in SharePoint Server 2010. In other
words, it appears that you have to use the same service account for the Web
application and service applications -- or else Excel Services simply doesn't
work (without some workaround).

I confirmed that changing the identity of the application pool in IIS for Excel
Services to be the same as the service account for Web application resolved the
problem. However, my preference was to keep the discrete service accounts -- if
at all possible.

Consequently, I added the service account for Excel Services to the SharePoint
content databases for the Web application. At first, I tried giving it "low
privilege" access, but I discovered this only resulted in different errors in
the event log when browsing to the dashboard page:

```Text
Source: Microsoft-SharePoint Products-SharePoint Foundation
Event ID: 5586
Description:
Unknown SQL Exception 262 occurred. Additional error information from SQL Server
is included below.

CREATE TABLE permission denied in database 'WSS_Content'.
```

and

```Text
Source: Microsoft-SharePoint Products-SharePoint Foundation
Event ID: 5617
Description:
There is a compatibility range mismatch between the Web server and database
"WSS_Content", and connections to the data have been blocked to due to this
incompatibility. This can happen when a content database has not been upgraded
to be within the compatibility range of the Web server, or if the database has
been upgraded to a higher level than the web server. The Web server and the
database must be upgraded to the same version and build level to return to
compatibility range.
```

While it's somewhat bewildering that Excel Services needs to create a table in
the SharePoint content database, I decided to just go ahead and give the service
account for Excel Services the same permissions to the content database as the
service account for the Web application (which, is to say, db\_owner). [Yeah, I
know, that's essentially the same as using a single service account for both the
Web application and SharePoint service applications. Perhaps someday the TFS
dashboards and Excel Services will "play nicely" together, even when configured
with restricted access.]

To add the the service account for SharePoint service applications to the
underlying content databases:

{{< deleted-block >}}

1. Click **Start**, point to **All Programs**, point to **Microsoft SQL Server
   2008**, and then click **SQL Server Management Studio**. The **Connect to
   Server** dialog box opens.
2. In the **Server type** list, click **Database Engine**.
3. Type the name of the server which hosts the SharePoint content databases, and
   then click **Connect**.
4. In **Object Explorer**, expand **Security**, and then expand **Logins**.
5. Right-click the login corresponding to the service account used for
   SharePoint service applications (**TECHTOOLBOX\svc-spserviceapp**) and then
   click **Properties**.
6. In the login properties dialog box,
   1. On the **User Mapping** page, in the **Users mapped to the login** list,
      click the checkbox for the SharePoint content database (**WSS\_Content**),
      and then in the database role membership list, click the checkboxes for
      the following roles:
      - **db\_owner**
      - **public**
   2. Repeat the previous step for any additional content databases that need to
      be accessed by Excel Services.
   3. Click **OK**.

{{< /deleted-block >}}

{{< div-block "note update" >}}

> **Update (2011-04-14)**
>
> I should have updated this post long ago based on the comment added by "todh2"
> regarding granting access to the database. Instead of using SQL Server
> Management Studio to configure permissions on the content database for the
> service account, use a little PowerShell to invoke the
> **[SPWebApplication.GrantAccessToProcessIdentity](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.administration.spwebapplication.grantaccesstoprocessidentity.aspx)**
> method:
>
> ```
> Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0
>
> $webApp = Get-SPWebApplication "http://cyclops"
>
> $webApp.GrantAccessToProcessIdentity("TECHTOOLBOX\svc-spserviceapp")
> ```
>
> Thanks to "todh2" for pointing this out.

{{< /div-block >}}

Once this configuration change is made, the "workbook cannot be opened" error no
longer occurs.
