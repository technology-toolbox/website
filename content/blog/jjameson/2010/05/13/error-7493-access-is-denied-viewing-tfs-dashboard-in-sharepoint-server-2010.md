---
title: "Error 7493 (\"Access is denied\") Viewing TFS Dashboard in SharePoint Server 2010"
date: 2010-05-13T12:43:00-06:00
excerpt: "Earlier this morning I was upgrading some of my Team Foundation Server (TFS) project sites (many of which were originally created with TFS 2005) in order to showcase the new dashboard features in TFS 2010. 
 While doing so, I encountered the following..."
aliases: ["/blog/jjameson/archive/2010/05/13/error-7493-access-is-denied-viewing-tfs-dashboard-in-sharepoint-server-2010.aspx"]
draft: true
categories: ["Development", "SharePoint"]
tags: ["TFS", "SharePoint 
			2010"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/05/13/error-7493-access-is-denied-viewing-tfs-dashboard-in-sharepoint-server-2010.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/05/13/error-7493-access-is-denied-viewing-tfs-dashboard-in-sharepoint-server-2010.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that
> blog ever goes away.

Earlier this morning I was upgrading some of my Team Foundation Server (TFS)
project sites (many of which were originally created with TFS 2005) in order
to showcase the new dashboard features in TFS 2010.

While doing so, I encountered the following error in the various Excel Web
Parts used to render the dashboards:

{{< blockquote "font-italic text-danger" >}}

An error occurred while accessing application id TFS from Secure Store Service.
The following connections failed to refresh:

TfsOlapReport

{{< /blockquote >}}

Looking at the event logs on the server, I found the following:

{{< log-excerpt >}}

<samp>Log Name: Application<br>
Source: Microsoft-SharePoint Products-Secure Store Service<br>
Date: 5/13/2010 6:11:25 AM<br>
Event ID: 7493<br>
Task Category: Secure Store<br>
Level: Error<br>
Keywords:<br>
User: TECHTOOLBOX\svc-spserviceapp<br>
Computer: cyclops.corp.technologytoolbox.com<br>
Description: The Microsoft Secure Store Service application Secure Store
Service failed to retrieve credentials. The error returned was 'Access is
denied.'. For more information, see the Microsoft SharePoint Products and
Technologies Software Development Kit (SDK). </samp>

{{< /log-excerpt >}}

I was initially perplexed by the "Access is denied" message since I was using
my administrator account to reconfigure the TFS project sites.

Diving into the SharePoint ULS logs, I discovered the underlying problem:

{{< log-excerpt >}}

```
05/13/2010 06:11:25.69 ... Secure Store Service ... ValidateCredentialClaims - Access Denied: Claims stored in the credentials did not match with the group claim for a group app. ...
05/13/2010 06:11:25.69 ... Secure Store Service ... The Microsoft Secure Store Service application Secure Store Service failed to retrieve credentials. The error returned was 'Access is denied.'. For more information, see the Microsoft SharePoint Products and Technologies Software Development Kit (SDK). ...
05/13/2010 06:11:25.69 ... Secure Store Service ... GetCredentials failed with the following exception: System.ServiceModel.FaultException`1[Microsoft.Office.SecureStoreService.Server.SecureStoreServiceFault]: Access is denied. (Fault Detail is equal to Microsoft.Office.SecureStoreService.Server.SecureStoreServiceFault). ...
```

{{< /log-excerpt >}}

It turns out my administrator account (TECHTOOLBOX\jjameson-admin) was not
in the group (TECHTOOLBOX\All Developers) that I originally specified when configuring
the credentials for the Secure Store target application for TFS. [The account
that I normally use to access the TFS project sites (i.e. TECHTOOLBOX\jjameson)
was in the group -- which explains why I hadn't seen the error before.]

After adding my administrator account to the group, the error no longer occurred
and the dashboards rendered as expected.

