---
title: "Avoid the Warning from Excel Services About Refreshing External Data"
date: 2010-05-19T03:42:00-06:00
excerpt: "In my post earlier this month on upgrading to Team Foundation Server (TFS) 2010 and SharePoint Server 2010 , I forgot to include the steps to change the Warn on Refresh setting of the trusted file location for the Excel Services Application. 
 If you..."
aliases: ["/blog/jjameson/archive/2010/05/18/avoid-the-warning-from-excel-services-about-refreshing-external-data.aspx", "/blog/jjameson/archive/2010/05/19/avoid-the-warning-from-excel-services-about-refreshing-external-data.aspx"]
draft: true
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "TFS", "SharePoint 2010"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/05/19/avoid-the-warning-from-excel-services-about-refreshing-external-data.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/05/19/avoid-the-warning-from-excel-services-about-refreshing-external-data.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

In my post earlier this month on [upgrading to Team Foundation Server (TFS) 2010 and SharePoint Server 2010](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010), I forgot to include the steps to change the **Warn on Refresh** setting of the trusted file location for the Excel Services Application.

If you leave this value set to the default (i.e. **Yes**), then you'll be greeted with the following message for each Excel Web Part on a TFS dashboard page:

{{< blockquote "font-italic" >}}

This workbook contains one or more queries that refresh external data. A malicious user can design a query to access confidential information and distribute it to other users or perform other harmful actions.

If you trust the source of this workbook, click Yes to enable queries to external data in this workbook. If you are not sure, click No so that changes are not applied to your workbook.

Do you want to enable queries to external data in this workbook?

{{< /blockquote >}}

To avoid warnings before refreshing external data in Excel Services:

1. On the SharePoint Central Administration home page, in the **Application Management** section, click **Manage service applications**.
2. On the **Service Applications** page, click **Excel Services Application**.
3. On the **Manage Excel Services Application** page, click **Trusted File Locations**.
4. On the **Excel Services Application Trusted File Locations** page, point to the trusted file location that you want to edit (e.g. **http://**), click the arrow that appears, and then click **Edit**.
5. On the **Excel Services Application Edit Trusted File Location** page, in the **External Data** section, clear the **Refresh warning enabled** checkbox, and then click **OK**.

Note that if you are using Microsoft Office SharePoint Server (MOSS) 2007, the configuration steps will be slightly different (the setting is changed through the SSP admin page). Refer to the following for more information:

{{< reference title="Manage Excel Services trusted file locations" linkHref="http://technet.microsoft.com/en-us/library/cc263009(office.12).aspx" >}}

