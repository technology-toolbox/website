---
title: "Upgrade Team Foundation Server 2008 to TFS 2010 (and SharePoint Server 2010)"
date: 2010-05-04T08:44:00-06:00
excerpt: "In my previous post , I provided an overview of the process of upgrading from TFS 2008 (and Windows SharePoint Services v3) to TFS 2010 (and SharePoint Server 2010). In this post, I provide more details about the upgrade process. 
 Note that if you are..."
aliases: ["/blog/jjameson/archive/2010/05/03/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010.aspx", "/blog/jjameson/archive/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010.aspx"]
draft: true
categories: ["My System", "Development", "SharePoint"]
tags: ["My System", "Visual Studio", "TFS", "SharePoint 2010"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

In my
[previous post](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010-overview),
I provided an overview of the process of upgrading from TFS 2008 (and Windows
SharePoint Services v3) to TFS 2010 (and SharePoint Server 2010). In this post,
I provide more details about the upgrade process.

Note that if you are not upgrading, but rather installing a new instance of TFS
2010 and SharePoint Server 2010, you can still follow the installation steps
below (supplementing steps from the TFS installation guide where appropriate).

The process below assumes the TFS environment is comprised of at least two
servers. The "data tier" only runs SQL Server (Database Engine and Analysis
Services), while the "application tier" runs essentially everything else (i.e.
TFS, SharePoint Server 2010, and Reporting Services), with the exception of Team
Foundation Build Service -- which is generally recommended to install on a
separate server.

### Plan the Installation

There are a number of configuration parameters that need to be determined before
installing Team Foundation Server 2010 and SharePoint Server 2010. The various
parameters and configuration settings for my environment are listed in the
following tables.

{{< table class="small" caption="Table 1 - Service Accounts" >}}

| <br>                    User Logon Name<br>                 | <br>                    Full Name<br>                 | <br>                    Description<br>                 |
| --- | --- | --- |
|  TECHTOOLBOX\svc-build  |  Service account for software builds  |  Used for the Team Foundation Build Service  |
|  TECHTOOLBOX\svc-sharepoint  |  Service account for SharePoint  |  Database access account used by SharePoint to connect to the configuration database (a.k.a. the SharePoint "farm account")  |
|  TECHTOOLBOX\svc-spserviceapp  |  Service account for SharePoint service applications  |  In SharePoint Server 2010, it is recommended to use a different account for running service applications (i.e. not the SharePoint "farm account")  |
|  TECHTOOLBOX\svc-sql  |  Service account for SQL Server  |   |
|  TECHTOOLBOX\svc-sql-agent  |  Service account for SQL Server Agent  |   |
|  TECHTOOLBOX\svc-tfs  |  Service account for Team Foundation Server  |   |
|  TECHTOOLBOX\svc-tfsreports  |  Service account for TFS reporting  |   |
|  TECHTOOLBOX\svc-web-tfs  |  Service account for Team Foundation Server Web site  |  Used for the application pool for the SharePoint Web application for TFS project sites  |

{{< /table >}}

{{< table class="small" caption="Table 2 - Domain Groups" >}}

| <br>                    Domain<br>                 | <br>                    Group Name<br>                 | <br>                    Group Scope<br>                 | <br>                    Group Type<br>                 | <br>                    Description<br>                 |
| --- | --- | --- | --- | --- |
|  TECHTOOLBOX  |  SharePoint Administrators  |  Global  |  Security  |   |
|  TECHTOOLBOX  |  SQL Server Administrators  |  Global  |  Security  |   |
|  TECHTOOLBOX  |  Team Foundation Server Administrators  |  Global  |  Security  |   |

{{< /table >}}

{{< table class="small"
caption="Table 3 - SharePoint Server 2010 Installation Parameters" >}}

| <br>                    Parameter<br>                 | <br>                    Value<br>                 |
| --- | --- |
|  SharePoint configuration database server  |  beast  |
|  SharePoint configuration database name  |  SharePoint\_Config  |
|  SharePoint Central Administration port number  |  22812  |
|  SharePoint Central Administration authentication provider  |  NTLM  |

{{< /table >}}

{{< table class="small" caption="Table 4 - Outgoing E-Mail Settings" >}}

| <br>                    Setting<br>                 | <br>                    Value<br>                 |
| --- | --- |
|  Outbound SMTP server  |  smtp.technologytoolbox.com  |
|  From address  |  svc-sharepoint@technologytoolbox.com  |
|  Reply-to address  |  no-reply@technologytoolbox.com  |
|  Character set  |  65001 (Unicode UTF-8)  |

{{< /table >}}

{{< table class="small" caption="Table 5 - SharePoint Web Applications" >}}

| <br>                    Name<br>                 | <br>                    Port<br>                 | <br>                    Host Header<br>                 | <br>                    Allow Anonymous<br>                 | <br>                    URL<br>                 | <br>                    App Pool Service Account<br>                 | <br>                    Database Name(s)<br>                 |
| --- | --- | --- | --- | --- | --- | --- |
|  SharePoint Central Administration v4  |  22812  |  (none)  |  No  |  http://cyclops:22812/  |  TECHTOOLBOX\svc-sharepoint  |  SharePoint\_AdminContent\_{GUID}  |
|  SharePoint - 80  |  80  |  (none)  |  No  |  http://cyclops/  |  TECHTOOLBOX\svc-web-tfs  | <ul><li>WSS_Content</li><li>WSS_Content_TFS (attached to Web application to restore existing TFS project sites)</li></ul> |

{{< /table >}}

{{< table class="small"
caption="Table 6 - Reporting Services Configuration Settings" >}}

| <br>                    Setting<br>                 | <br>                    Value<br>                 |
| --- | --- |
|  Instance Configuration  |  Default instance (MSSQLSERVER)  |
|  Service Account  |  NT AUTHORITY\NETWORK SERVICE  |
|  SMTP Settings - Sender Address  |  svc-tfsreports@technologytoolbox.com  |
|  SMTP Settings - SMTP Server  |  smtp.technologytoolbox.com  |

{{< /table >}}

{{< table class="small"
caption="Table 7 - Secure Store Target Application Settings for TFS Dashboards" >}}

| <br>                    Setting<br>                 | <br>                    Value<br>                 |
| --- | --- |
|  Target Application ID  |  TFS  |
|  Display Name  |  Team Foundation Server  |
|  Contact E-mail  |  jjameson@technologytoolbox.com  |
|  Target Application Type  |  Group  |
|  Members  |  TECHTOOLBOX\All Developers  |

{{< /table >}}

{{< table class="small" caption="Table 8 - TFS Configuration Settings" >}}

| <br>                    Setting<br>                 | <br>                    Value<br>                 |
| --- | --- |
|  Databases - SQL Server Instance  |  BEAST  |
|  Databases - Configuration database to upgrade  |  TfsIntegration  |
|  Account - Service Account - Account Name  |  TECHTOOLBOX\svc-tfs  |
|  Account - Authentication Method  |  NTLM  |
|  Application Tier - Web Site Name  |  Team Foundation Server  |
|  Application Tier - Port  |  8080  |
|  Application Tier - IIS Virtual Directory  |  tfs  |
|  Reporting - Cofiguration Reporting for use with Team Foudnation Serve  |  (checked)  |
|  Reporting - Reporting Services - Reporting Services Instance  |  cyclops  |
|  Reporting - Reporting Services - Report Server URL  |  [http://CYCLOPS:80/ReportServer](http://CYCLOPS:80/ReportServer)  |
|  Reporting - Reporting Services - Report Manager URL  |  [http://CYCLOPS:80/Reports](http://CYCLOPS:80/Reports)  |
|  Reporting - Analysis Services - SQL Server Analysis Services Instance  |  BEAST  |
|  Reporting - Report Reader Account - Use a different account than the Team Foundation Server service account for the report reader account  |  (checked)  |
|  Reporting - Report Reader Account - Account Name  |  TECHTOOLBOX\svc-tfsreports  |
|  SharePoint Products - Configuration SharePoint for use with Team Foundation Server  |  (checked)  |
|  SharePoint Products - Settings - Site URL  |  [http://cyclops/sites](http://cyclops/sites)  |
|  SharePoint Products - Settings - Administration URL  |  [http://cyclops:22812/](http://cyclops:22812/)  |
|  Project Collection - Name for the new collection  |  DefaultCollection  |
|  Project Collection - Description  |  Project(s) upgraded from previous Team Foundation Server  |

{{< /table >}}

{{< table class="small"
caption="Table 9 - Team Foundation Build Configuration Settings" >}}

| <br>                    Setting<br>                 | <br>                    Value<br>                 |
| --- | --- |
|  Team Foundation Build Server  |  DAZZLER  |
|  Project Collection - Team project collection to configure build services for  |  http://cyclops:8080/tfs/DefaultCollection  |
|  Build Services - Number of build agents to run on this build machine  |  1  |
|  Settings - Run Team Foundation Build Service as  |  TECHTOOLBOX\svc-build  |
|  Settings - Accept connections from Team Foundation Server on  |  Default port 9191  |

{{< /table >}}

### Deploy and Configure the Server Infrastructure

#### Install Windows Server 2008 R2

Install Windows Server 2008 R2 according to your corporate standards. Ensure
that all standard programs, such as antivirus and server monitoring utilities,
are installed and configured.

#### Create service accounts

To create the necessary service accounts for SharePoint Server 2010 and TFS
2010:

1. Start the **Active Directory Users and Computers** console.
2. Under the domain node in the tree, select the node for the organizational
   unit where the service account should be created (e.g. **IT\Service
   Accounts**).
3. On the **Action** menu, point to **New**, and then click **User**.
4. In the **New Object - User**dialog:
   1. Enter the information from
      [Table 1](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010).
   2. Click **Next**.
   3. Clear the **User must change password** at next logon check box.
   4. Select the **User cannot change password** check box.
   5. Select the **Password never expires** check box.
   6. Click **Next**.
   7. Click **Finish**.
5. Repeat steps 3 and 4 to create the remaining service accounts listed in
   [Table 1](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010).

#### Create domain groups

To create the domain groups for SharePoint Server 2010 and TFS 2010:

1. If necessary, start the **Active Directory Users and Computers** console.
2. Under the domain node in the tree, select the node for the OU where the group
   should be created (e.g. **IT\Groups**).
3. On the **Action** menu, point to **New**, and then click **Group**.
4. In the **New Object - Group**dialog:
   1. Enter the information from
      [Table 2](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010).
   2. Click **OK**.
5. Repeat steps 3 and 4 to create the remaining groups listed in
   [Table 2](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010).
6. Add the appropriate users to each domain group created in this step.
7. Close the **Active Directory Users and Computers** console.

### Back Up Team Foundation Server Data

As prescribed in the TFS installation guide, back up all TFS data -- including
the TFS databases, SharePoint databases, Reporting Services encryption key, and
the TFS Web.config file.

### Install/Upgrade SQL Server 2008 on Data Tier

Using the configuration steps provided in the TFS installation guide (**How to:
Install SQL Server 2008**), install SQL Server 2008 (Database Engine and
Analysis Services) on the TFS database server (BEAST). Then run Windows Update
to install SQL Server 2008 Service Pack 1.

In order to install SharePoint Server 2010, you must also install SQL Server
2008 SP1 CU2 (or later):

{{< reference
title="Cumulative update package 2 for SQL Server 2008 Service Pack 1"
linkHref="http://support.microsoft.com/kb/970315" >}}

### Install SharePoint Server 2010

Using the configuration steps provided in the following TechNet article, install
SharePoint Server 2010 using the installation parameters specified in
[Table 3](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010):

{{< reference
title="Deploy a single server with SQL Server (SharePoint Server 2010)"
linkHref="http://technet.microsoft.com/en-us/library/cc262243(office.14).aspx" >}}

Complete the deployment procedures in the following sections:

1. Run the Microsoft SharePoint Products Preparation Tool
2. Run Setup
3. Run the SharePoint Products Configuration Wizard

> **Important**
>
> Do not run the Farm Configuration Wizard (the TFS Web application and service
> applications will be configured in the following steps).

#### Add SharePoint Central Administration to the Local intranet zone

Adding the Central Administration site to the **Local intranet** zone (and using
the default settings for this zone) enables single sign-on when accessing the
site. This is discussed in more detail in the following blog post:

{{< reference title="Be \"In the Zone\" to Avoid Entering Credentials"
linkHref="/blog/jjameson/2007/03/22/be-in-the-zone-to-avoid-entering-credentials"
linkText="https://www.technologytoolbox.com/blog/jjameson/archive/2007/03/22/be-in-the-zone-to-avoid-entering-credentials.aspx" >}}

#### Add the SharePoint bin folder to the PATH environment variable

Append **C:\Program Files\Common Files\Microsoft Shared\web server
extensions\14\BIN** to the **PATH** environment variable (in order to run
stsadm.exe from various folder locations without having to specify the full
path).

#### Add the SharePoint administrators group to the WSS\_ADMIN\_WPG group and SharePoint\_Shell\_Access database role

To add the SharePoint administrators group to the local WSS\_ADMIN\_WPG group,
on the SharePoint server:

1. Click **Start**, point to **Administrative Tools**, and then click **Server
   Manager**.
2. In the **Server Manager** console, expand **Configuration**, expand **Local
   Users and Groups**, and then click **Groups**.
3. In the list of groups, right-click **WSS\_ADMIN\_WPG** and then click
   **Properties**.
4. In the **WSS\_ADMIN\_WPG Properties** dialog box, click **Add**.
5. In the Select Users, Computers, Service Accounts or Groups dialog box, type
   the name of the SharePoint administrators group specified in
   [Table 2](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010),
   click **Check Names**, and then click **OK**.
6. Click **OK** to save the changes to the group.

To add the SharePoint administrators group to the database role, on the database
server:

1. Click **Start**, point to **All Programs**, point to **Microsoft SQL Server
   2008**, and then click **SQL Server Management Studio**. The **Connect to
   Server** dialog box opens.
2. In the **Server type** list, click **Database Engine**.
3. Type the name of the server which hosts the configuration database, and then
   click **Connect**.
4. In **Object Explorer**, expand **Security**, and then expand **Logins**.
5. If the SharePoint administrators group does not exist, right-click **Logins**
   and click **New login**. If the SharePoint administrator group already has a
   corresponding login for SQL Server, right-click the login and then click
   **Properties**.
6. In the login properties dialog box:
   1. On the **General** page, in the **Login name** box, type the name of the
      SharePoint administrators group from
      [Table 2](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010)
      using the form {DOMAIN}\{group name}.
   2. On the **User Mapping** page, in the **Users mapped to the login** list,
      click the checkbox next to the SharePoint configuration database (from
      Table 3), and then in the database role membership list, click the
      checkbox for **SharePoint\_Shell\_Access**.
   3. Click **OK**.

#### Configure mail services

Using the configuration steps provided in the following TechNet article,
configure the outgoing e-mail settings for SharePoint using the values specified
in
[Table 4](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010):

{{< reference title="Configure outgoing e-mail (SharePoint Server 2010)"
linkHref="http://technet.microsoft.com/en-us/library/cc263462(office.14).aspx" >}}

#### Create SharePoint Web application

Using the configuration steps provided in the following TechNet article, create
a new Web application using the configuration settings specified in
[Table 5](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010):

{{< reference title="Create a Web application (SharePoint Server 2010)"
linkHref="http://technet.microsoft.com/en-us/library/cc261875(office.14).aspx" >}}

Note that the **SharePoint Central Administration v4** Web application listed in
[Table 5](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010)
was already created by the SharePoint Products Configuration Wizard.

#### Grant DCOM permissions on IIS WAMREG admin Service

In order to avoid errors in the Windows event log (e.g. Event ID 10016), grant
the **WSS\_ADMIN\_WPG** and **WSS\_WPG** group appropriate permissions on the
IIS WAMREG admin Service, as described in
[KB 920783](http://support.microsoft.com/kb/920783). This is discussed in more
detail in the following blog post:

{{< reference title="Event ID 10016, KB 920783, and the WSS_WPG Group"
linkHref="/blog/jjameson/2009/10/17/event-id-10016-kb-920783-and-the-wss-wpg-group"
linkText="https://www.technologytoolbox.com/blog/jjameson/archive/2009/10/17/event-id-10016-kb-920783-and-the-wss-wpg-group.aspx" >}}

{{< deleted-block >}}

### Fix assembly name in /\_controltemplates/TaxonomyPicker.ascx

In order to avoid errors in the Windows event log (e.g. Source: SharePoint
Foundation, Event ID: 7043), edit the TaxonomyPicker.ascx file to fix the
assembly specified in the Control directive.

To fix the TaxonomyPicker.ascx file:

1. Click **Start**, point to **All Programs**, point to **Accessories**, and
   right-click **Command Prompt**, and then click **Run as administrator**.

2. At the command prompt, type the following command:
   
   {{< console-block-start >}}
   
   notepad "C:\Program Files\Common Files\Microsoft Shared\Web Server
   Extensions\14\TEMPLATE\CONTROLTEMPLATES\TaxonomyPicker.ascx"
   
   {{< console-block-end >}}

3. In Notepad, in the assembly specified in the **Control** directive, replace
   the **"&#44;**" (without the quotes) with a comma (',') and then save the
   file.

{{< /deleted-block >}}

> **Update (2011-04-14)**
>
> The TaxonomyPicker.ascx file is fundamentally broken in SharePoint 2010.
> Instead of trying to fix the assembly name, just rename the file (since it
> apparently isn't used by SharePoint).

### Rename TaxonomyPicker.ascx

In order to avoid errors in the Windows event log (e.g. Source: SharePoint
Foundation, Event ID: 7043), rename the out-of-the-box TaxonomyPicker.ascx file.

> **Important**
>
> This task must be completed on each SharePoint server in the farm.

To rename the TaxonomyPicker.ascx file:

1. Open Windows Explorer and browse to the following folder:
   
   **C:\Program Files\Common Files\Microsoft Shared\Web Server
   Extensions\14\TEMPLATE\CONTROLTEMPLATES**

2. Right-click **TaxonomyPicker.ascx**, click **Rename**, and then change the
   filename to **TaxonomyPicker.ascx\_broken**.

> **Note**
>
> Changing the file extension causes the problematic file to be skipped by
> ASP.NET when compiling the controls in the folder.

### Install SQL Server 2008 Reporting Services

#### Install Reporting Services on the TFS application server

Using the configuration steps provided in the TFS installation guide (**How to:
Install SQL Server 2008**), install Reporting Services on the TFS application
server (CYCLOPS) using the configuration settings specified in
[Table 6](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010).

#### Install SQL Server 2008 Service Pack 1 on the TFS application server

Run Windows Update on the TFS application server (CYCLOPS) to install the
recommended patches (including SQL Server 2008 SP1 -- for Reporting Services)
and, if necessary, reboot the server.

#### Configure Reporting Services

If upgrading to TFS 2010 from a previous version, skip this step (Reporting
Services is configured later when restoring data).

If performing a new installation of TFS 2010 (not upgrading from a previous
version), then use the configuration steps provided in the TFS installation
guide to configure Reporting Services. Verify the Reporting Services
configuration has been restored successfully by browsing to the Report Manager
URL (e.g. [http://cyclops/Reports](http://cyclops/Reports)).

### Configure SharePoint Server 2010 for TFS Dashboards

The steps for configurating SharePoint Server 2010 for TFS dashboards are
provided in following blog post:

{{< reference
title="Configuring SharePoint Server 2010 for Dashboard Compatibility with TFS 2010"
linkHref="http://blogs.msdn.com/team_foundation/archive/2010/03/06/configuring-sharepoint-server-2010-beta-for-dashboard-compatibility-with-tfs-2010-beta2-rc.aspx" >}}

Note that the blog post is based on the Beta 2 version of SharePoint Server 2010
and there were a few changes in the RTM version.

To start the Excel Calculation Services and Secure Store Service in SharePoint
Server 2010:

1. On the SharePoint Central Administration home page, click **Configuration
   Wizards**.
2. On the **Configuration Wizards** page, in the **Farm Configuration** section,
   click **Launch the Farm Configuration Wizard**.
3. On the **Configure your SharePoint Farm**page:
   1. Click **Start the Wizard**.
   2. In the **Service Account** section, ensure the option to **Create a new
      managed account** is selected, in the **User name** box, type the service
      account for SharePoint service applications listed in
      [Table 1](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010),
      and type the corresponding password in the **Password** box. In the
      **Services** section, clear the checkboxes for all of the services except
      **Excel Services Application** and **Secure Store Service**. Click
      **Next**.
   3. If prompted to create a top-level Web site, click **Skip**.
   4. Click **Finish**.

To avoid warnings before refreshing external data in Excel Services:

1. On the SharePoint Central Administration home page, in the **Application
   Management** section, click **Manage service applications**.
2. On the **Service Applications** page, click **Excel Services Application**.
3. On the **Manage Excel Services Application** page, click **Trusted File
   Locations**.
4. On the **Excel Services Application Trusted File Locations** page, point to
   the trusted file location that you want to edit (e.g. **http://**), click the
   arrow that appears, and then click **Edit**.
5. On the **Excel Services Application Edit Trusted File Location** page, in the
   **External Data** section, clear the **Refresh warning enabled** checkbox,
   and then click **OK**.

To configure the Secure Store Service for TFS dashboards:

1. On the SharePoint Central Administration home page, in the **Application
   Management** section, click **Manage service applications**.
2. On the **Service Applications** page, click **Secure Store Service**.
3. If a key has not been generated before:
   1. In the **Key Management** group of the Ribbon, click **Generate New Key**.
   2. On the **Generate New Key** page, type a pass phrase string in the **Pass
      Phrase** box, type the same string in the **Confirm Pass Phrase** box, and
      then click **OK**.
4. In the **Manage Target Applications** group, click **New**.
5. On the **Create New Secure Store Target Application** page, enter the target
   application settings from
   [Table 7](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010)
   and then click **Next**.
6. On the **Specify credential fields for your Secure Store Target Application**
   page, click **Next** to accept the default fields (**Windows User Name** and
   **Windows Password**).
7. On the **Specify the membership settings** page, in the **Target Application
   Administrators** field, add the TFS administrators group specified in
   [Table 2](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010).
   In the **Members** field, enter the group(s) from
   [Table 7](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010).
   Click **OK** to complete configuring the target application.

To set credentials for the TFS target application:

1. On the **TFS** target application, point to the target application
   identifier, click the arrow that appears, and then, in the menu, click **Set
   credentials**.
2. On the **Set Credentials for Secure Store Target Application (Group)** page,
   enter the credentials for the service account for TFS reporting from
   [Table 1](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010),
   and click **OK**.

### Add TFS service account to SharePoint Farm Administrators group

To add the TFS service account to the SharePoint Farm Administrators group:

1. On the SharePoint Central Administration home page, in the **Security**
   section, click **Manage the farm administrators group**.
2. On the **People and Groups - Farm Administrators** page, in the toolbar, on
   the **New** menu, click **Add Users**.
3. In the **Grant Permissions** dialog box, in the **Users/Groups** box, type
   the TFS service account listed in
   [Table 1](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010),
   clear the **Send welcome e-mail to the new users** checkbox, and click
   **OK**.

### Restore Data

#### Restore SQL Server Databases

If necessary, restore the TFS and SharePoint databases from the backups taken at
the beginning of the upgrade process.

#### Configure Reporting Services

To connect to the existing ReportServer and ReportServerTempDB databases on the
TFS database server:

1. Click **Start**, point to **All Programs**, point to **Microsoft SQL Server
   2008**, point to **Configuration Tools**, and click **Reporting Services
   Configuration Manager**. If prompted by **User Account Control** to allow the
   program to make changes to this computer, click **Yes**.
2. The **Reporting Services Configuration Connection** dialog box appears.
3. In the **Server Name** box, ensure the TFS application server is specified.
   In **Report Server Instance**, ensure the correct instance is specified.
   Click **Connect**.
4. On the **Reporting Services Configuration Manager** page, click **Start** if
   the Report Service status reads **Stopped**.
5. In the navigation bar, click **Web Service URL**.
6. On the **Web Service URL** page, click **Apply** to accept the default values
   in the **Virtual Directory**, **IP Address**, and **TCP Port** boxes.
7. In the navigation bar, click **Database**.
8. On the **Report Server Database** page, click **Change Database**.
9. The **Report Server Database Configuration Wizard** appears.
10. On the **Action** page of the wizard, click **Choose an existing report
    server database**, and click **Next**.
11. On the **Database Server** page of the wizard, type the name of the TFS
    database server (BEAST) in **Server Name**, and then click **Test
    Connection**. Confirm the test connection succeeded by clicking **OK**, and
    then click **Next.**
12. On the **Database** page of the wizard, in the **Report Server Database**
    dropdown list, select **ReportServer**, and then click **Next**.
13. On the **Credentials** page of the wizard, click **Next** to accept the
    default values in the **Authentication Type**, **User name**, and
    **Password** boxes (i.e. **Service Credentials** and **NT AUTHORITY\NETWORK
    SERVICE**).
14. On the **Summary** page of the wizard, verify the information, and click
    **Next**.
15. On the **Progress and Finish** page of the wizard, confirm the configuration
    was successful, and then click **Finish**.
16. In the navigation bar for **Reporting Services Configuration Manager**,
    click **Report Manager URL**.
17. On the **Report Manager URL** page, click **Apply** to accept the default
    value in the **Virtual Directory** box.
18. In the navigation bar for **Reporting Services Configuration Manager**,
    click **E-mail Settings**.
19. On the **E-mail Settings** page, enter the configuration settings from
    [Table 6](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010),
    and then click **Apply**.
20. In the navigation bar for **Reporting Services Configuration Manager**,
    click **Encryption Keys**.
21. On the **Encryption Keys** page, click **Restore**.
22. The **Restore Encryption Key** window appears.
23. Locate the encryption key backup file created previously, type the
    corresponding password, and then click **OK**.
24. In the **Reporting Services Configuration Manager**, click **Exit**.

#### Verify the Reporting Services configuration

To verify that the Reporting Services configuration has been restored
successfully, browse to the Report Manager URL (e.g.
[http://cyclops/Reports](http://cyclops/Reports)) to confirm the TFS reports
appear as expected.

You may encounter the following error:

{{< blockquote "font-italic text-danger" >}}

The feature: "Scale-out deployment" is not supported in this edition of
Reporting Services. (rsOperationNotSupported)

{{< /blockquote >}}

To resolve this error, remove the duplicate server using one of the following
methods:

- Start the **Reporting Services Configuration Manager**, connect to the
  Reporting Services instance, and then in the navigation bar, click **Scale-out
  Deployment**. On the **Scale-out Deployment** page, select the duplicate
  server instance and then click **Remove Server**.
- Use the
  [rskeymgmt utility](http://msdn.microsoft.com/en-us/library/ms162822.aspx) to
  identify and remove the duplicate server (by first specifying the {{< kbd
  "-l" >}} command-line option, and then specifying the {{< kbd "-r" >}}
  command-line option, as described in the first comment on the following MSDN
  article:
  [Moving the Report Server Databases to Another Computer](http://msdn.microsoft.com/en-us/library/ms156421.aspx)).

Confirm that the error no longer occurs when browsing to the Report Manager URL
and the TFS reports appear as expected.

#### Attach SharePoint content database with TFS project sites

Attach the content database with the existing TFS project sites (from the
previous version of SharePoint) to the new SharePoint farm.

To attach the SharePoint content database by using Windows Powershell:

1. On the **Start** menu, click **All Programs**, click **Microsoft SharePoint
   2010 Products**, right-click **SharePoint 2010 Management Shell**, and then
   click **Run as administrator**. If prompted by **User Account Control** to
   allow the program to make changes to this computer, click **Yes**.

2. At the Windows PowerShell command prompt, type the following command:
   
   {{< console-block-start >}}
   
   Mount-SPContentDatabase -Name &lt;DatabaseName&gt; -DatabaseServer
   &lt;ServerName&gt; -WebApplication &lt;URL&gt; [-Updateuserexperience]
   
   {{< console-block-end >}}
   Where:
   
   - <var>&lt;DatabaseName&gt;</var> is the name of the database you want to
     upgrade.
   - <var>&lt;ServerName&gt;</var> is server on which the database is stored.
   - <var>&lt;URL&gt;</var> is the URL for the Web application that will host
     the sites.
   - **-Updateuserexperience** specifies to update the sites with the new
     SharePoint user experience (part of Visual Upgrade). If you omit this
     parameter, the sites retain the old user experience after upgrade.
   
   For example:
   
   {{< console-block-start >}}
   
   Mount-SPContentDatabase -Name WSS\_Content\_TFS -DatabaseServer BEAST
   -WebApplication http://cyclops -Updateuserexperience
   
   {{< console-block-end >}}

More information on attaching SharePoint content databases is provided in the
following TechNet article:

{{< reference title="Attach databases and upgrade to SharePoint Server 2010"
linkHref="http://technet.microsoft.com/en-us/library/cc263299(office.14).aspx" >}}

Note that if you browse to a TFS project site at this point, an error is
displayed on the home page in the **Remaining Work** Web Part, as shown below.
This occurs because the TFS extensions for SharePoint have not been installed
yet.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Upgraded-TFS-project-site-with-RS-error-600x481.png"
alt="Upgraded TFS project site in SharePoint Server 2010 (with Reporting
Services error)"
class="screenshot" height="481" width="600"
title="Figure 1: Upgraded TFS project site in SharePoint Server 2010 (with
Reporting Services error)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Upgraded-TFS-project-site-with-RS-error-988x792.png)

There are also a couple of other issues with the upgraded TFS project site:

- The home page is customized (i.e. "unghosted") and consequently must be reset
  (i.e. "reghosted") in order to render using the new SharePoint experience
  (i.e. the "v4 master page").
- Assuming the TFS project site was originally created with Windows SharePoint
  Services v2 (e.g. in TFS 2005), many of the global navigation elements are now
  broken. For example, the **Site Settings** link shown in the previous figure
  refers to **\_layouts/1033/settings.aspx** -- which was a valid page in WSS v2
  and WSS v3. However, in SharePoint Server 2010, the URL of the Site Settings
  page is **\_layouts/settings.aspx**.

To reset the home page for a TFS project site:

1. Browse to the Site Settings page for the site (e.g.
   [http://cyclops/sites/Demo/\_layouts/settings.aspx](http://cyclops/sites/Demo/_layouts/settings.aspx)).
2. On the **Site Settings** page, in the **Site Actions** section, click **Reset
   to site definition**.
3. On the **Reset Page to Site Definition Version** page, click the option to
   **Reset all pages in this site to site definition version**, and then click
   **Reset**.

> **Tip**
>
> If you need to do this for a number of sites, you should consider using
> PowerShell instead, as described in the following blog post:
>
> {{< reference
> title="Use PowerShell to \"Reset to Site Definition\" in SharePoint Server
> 2010"
> linkHref="/blog/jjameson/2010/05/18/use-powershell-to-quot-reset-to-site-definition-quot-in-sharepoint-server-2010"
> linkText="https://www.technologytoolbox.com/blog/jjameson/archive/2010/05/18/use-powershell-to-quot-reset-to-site-definition-quot-in-sharepoint-server-2010.aspx"
>
> > }}

To remove obolete links from the top link bar of a TFS project site:

1. Browse to the Site Settings page for the site (e.g.
   [http://cyclops/sites/Demo/\_layouts/settings.aspx](http://cyclops/sites/Demo/_layouts/settings.aspx)).
2. On the **Site Settings** page, in the **Look and Feel** section, click **Top
   link bar**.
3. On the **Top Link Bar** page, click the edit icon next to the navigation
   item.
4. On the **Edit Navigation Link** page, click **Delete**. When prompted to
   confirm deleting the link, click **OK**.
5. Repeat the previous two steps for any other links you want to remove.

### Install Team Foundation Server

Using the steps provided in the TFS installation guide (**How to: Install Team
Foundation Server**), install TFS on the application server (CYCLOPS).

### Upgrade Team Foundation Server

Using the steps provided in the TFS installation guide (**How to: Upgrade Team
Foundation Server Using the Team Foundation Server Configuration Tool**),
upgrade TFS using the parameters specified in
[Table 8](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010).

### Verify upgraded TFS project sites

Browse to one of the upgraded TFS project sites to confirm the **Remaining
Work** Web Part on the home page renders as expected (now that the TFS
extensions for SharePoint have been installed).

Note that a different error may be shown in the **Remaining Work** Web Part:

{{< blockquote "font-italic text-danger" >}}

Reporting Services Error

***


An error has occurred during report processing. (rsProcessingAborted) Get Online
Help
Query execution failed for dataset 'DefaultIterationParam'.
(rsErrorExecutingCommand) Get Online Help
For more information about this error navigate to the report server on the local
server machine, or enable remote errors

***


SQL Server Reporting Services

{{< /blockquote >}}

This error typically occurs when the TFS data warehouse (i.e. the OLAP cube) has
not yet been processed. Wait for the warehouse database to be updated.

Refer to the following blog post for some helpful reports that display details
about TFS warehouse jobs:

{{< reference title="TFS2010: Warehouse and Job Service Administrator Reports"
linkHref="http://blogs.msdn.com/granth/archive/2010/02/07/tfs2010-warehouse-and-job-status-reports.aspx" >}}

Once the warehouse database has been updated, an upgraded TFS project site
should appear similar to the following:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Upgraded-TFS-project-site-445x600.png"
alt="Upgraded TFS project site in SharePoint Server 2010" class="screenshot"
height="600" width="445"
title="Figure 2: Upgraded TFS project site in SharePoint Server 2010" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Upgraded-TFS-project-site-987x1332.png)

> **Tip**
>
> To significantly improve the appearance of the upgraded TFS project home page,
> edit the **Remaining Work** Web Part and set the **Height** to **600 pixels**
> and the **Width** to **900 pixels**.

For more information on updating a team project, refer to the following MSDN
article:

{{< reference title="Updating an Upgraded Team Project to Access New Features"
linkHref="http://msdn.microsoft.com/en-us/library/ff432837.aspx" >}}

#### Add service account for SharePoint service applications to content databases

There appears to be a bug in SharePoint Server 2010 when the service account for
the Web application is different from the service account for the service
applications (e.g. Excel Calculation Services and the Secure Store Service).
This is discussed in more detail in the following blog post:

{{< reference
title="\"The workbook cannot be opened\" Error with SharePoint Server 2010 (and TFS 2010)"
linkHref="/blog/jjameson/2010/05/04/the-workbook-cannot-be-opened-error-with-sharepoint-server-2010-and-tfs-2010"
linkText="https://www.technologytoolbox.com/blog/jjameson/archive/2010/05/04/the-workbook-cannot-be-opened-error-with-sharepoint-server-2010-and-tfs-2010.aspx" >}}

To avoid this error, add the the second service account to the underlying
content databases. ~~On the database server:~~

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
7. Repeat the previous step for any additional content databases that need to
   be accessed by Excel Services (**WSS\_Content\_TFS**).
   3. Click **OK**.

{{< /deleted-block >}}

> **Update (2011-04-14)**
>
> ```
> Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0
>
> $webApp = Get-SPWebApplication "http://cyclops"
>
> $webApp.GrantAccessToProcessIdentity("TECHTOOLBOX\svc-spserviceapp")
> ```

#### Configure enterprise application definition for TFS dashboards

Using the configuration steps provided in the TFS installation guide
(**Configure the Enterprise Application Definition**), configure the Secure
Store target application for TFS dashboards using the **Target Application ID**
specified in
[Table 7](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010).

### Install Team Explorer 2010 or Visual Studio 2010

In order to create a new TFS project, you must install either Team Explorer 2010
or Visual Studio 2010. If you want to be able to create new team projects (or
browse existing projects) directly from the TFS application server, install Team
Explorer 2010.

> **Important**
>
> If you need to access TFS 2010 from a VSTS 2008 client (for example, to
> continue to use the source control integration features in Expression Web 3),
> you must download and install an update:
>
> {{< reference
> title="Visual Studio Team System 2008 Service Pack 1 Forward Compatibility
> Update for Team Foundation Server 2010 (Installer)"
> linkHref="http://www.microsoft.com/downloads/details.aspx?displaylang=en&FamilyID=cf13ea45-d17b-4edc-8e6c-6c5b208ec54d"
>
> > }}
>
> Refer to [KB 974558](http://support.microsoft.com/?kbid=974558) for more
> information on the compatibility update.

### Connect to TFS

To connect to the upgraded TFS instance, you need to change the URL specified in
Visual Studio from the TFS 2008 format (e.g.
[http://cyclops:8080](http://cyclops:8080)) to the TFS 2010 format (e.g.
[http://cyclops:8080/tfs](http://cyclops:8080/tfs)).

If connecting from a Visual Studio 2008 client (with the compatibility update
installed), you must also specify the TFS project collection in the URL (e.g.
[http://cyclops:8080/tfs/DefaultCollection](http://cyclops:8080/tfs/DefaultCollection)).

### Create a new team project in TFS

To verify the TFS upgrade, create a new team project in TFS (for example, a new
project named **Test**) using the **MSF for Agile Software Development v5.0**
process template.

> **Note**
>
> In order to create a new team project (with the default project options) using
> Team Explorer on a server that hosts SharePoint Server 2010 and SQL Server
> Reporting Services, you need to run Visual Studio as an administrator.

For more information on creating a new TFS project, refer to the following:

{{< reference title="Create a Team Project"
linkHref="http://msdn.microsoft.com/en-us/library/ms181477.aspx" >}}

### Install TFS Power Tools

To enable additional check-in policies and other access additional tools (TFS
2010 Best Practices Analyzer), download and install the new version of the
Visual Studio Power Tools for TFS 2010:

{{< reference title="Visual Studio Power Tools"
linkHref="http://msdn.microsoft.com/en-us/vstudio/bb980963.aspx" >}}

### Install Team Foundation Build

Using the steps provided in the TFS installation guide (**How to: Install Team
Foundation Build Service**), install the Team Foundation Build Service on the
build server (DAZZLER) using the parameters specified in
[Table 9](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010).

