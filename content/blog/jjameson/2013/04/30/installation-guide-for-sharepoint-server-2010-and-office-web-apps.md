---
layout: single
title: Installation Guide for SharePoint Server 2010 and Office Web Apps
date: 2013-04-30T03:12:32-06:00
lastmod: 2013-04-30T03:23:52-06:00
description:
  This post provides a sample installation guide for an extranet platform based
  on SharePoint Server 2010 and Office Web Apps.
aliases:
  [
    "/blog/jjameson/archive/2013/04/29/installation-guide-for-sharepoint-server-2010-and-office-web-apps.aspx",
    "/blog/jjameson/archive/2013/04/30/installation-guide-for-sharepoint-server-2010-and-office-web-apps.aspx",
  ]
categories: ["My System", "SharePoint"]
tags: ["My System", "SharePoint 2010"]
attachment:
  url: "https://assets.technologytoolbox.com/blog/jjameson/Documents/Fabrikam Demo - SharePointExtranet.zip"
  fileName: Fabrikam Demo - SharePointExtranet.zip
  fileSizeInBytes: 3704288
---

This post provides a sample installation guide for an extranet platform based on
SharePoint Server 2010 and Office Web Apps. As noted in
[my previous post](/blog/jjameson/2012/03/17/always-create-installation-guides-for-predictable-and-repeatable-deployments),
much of this content is included on TechNet (in fact, a good portion of this
installation guide has roots in the TechNet documentation for Microsoft Office
SharePoint Server 2007). However, this guide augments the SharePoint 2010
TechNet documentation with corrections and additional content to fix omissions.
It also includes details specific to a particular environment (e.g. service
account names and database sizes).

For the sake of understanding the context of this installation guide, imagine
being tasked with deploying a new extranet solution for Fabrikam Technologies
(my favorite fictitious manufacturing company). The extranet solution needs to
be available as soon as possible and thus various capabilities will be released
over a series of short sprints (or _iterations_, if you prefer that term
instead). The scope for the first sprint is limited to the following:

- Site branding (minimal customization to meet Fabrikam corporate standards)
- Claims authentication for employees, partners, suppliers, and resellers
- Collaboration sites used to share documents, contact details, and other
  information with Fabrikam partners
- Office Web Apps to enable people to view, create, and edit Office documents
- Search (initial configuration)

---

{{< article-block-start "document numbered-headings" >}}

# Introduction

This document explains how to install and configure the Fabrikam Extranet.

## Purpose

This installation guide provides the detailed guidance and step-by step
procedures necessary to deploy the new extranet solution for Fabrikam
([extranet.fabrikam.com](http://extranet.fabrikam.com)), which is built on
Microsoft SharePoint Server 2010 and uses Office Web Apps to provide a
collaboration environment for Fabrikam employees, partners, suppliers, and
resellers.

Installation guides provide an essential reference for efficiently deploying
solutions in a consistent, repeatable fashion. Like the checklists that even the
most experienced pilots utilize before every takeoff, this installation guide
minimizes the risk of accidentally omitting a critical task.

## Audience

This document is primarily intended for the Release/Operations role for
deploying to the Production environment (PROD). However it is also expected to
be used by the Test team when configuring the Test environment (TEST), as well
as the Development team when configuring the Development integration environment
(DEV) and local development VMs.

## References

- Vision/Scope – Fabrikam Extranet
- Infrastructure Model - Fabrikam Extranet
- SQL Server Installation Guide - Fabrikam Extranet
- {{< reference
  title="Deployment scenario: Multiple servers for a three-tier farm (SharePoint Server 2010)"
  linkHref="http://technet.microsoft.com/en-us/library/ee805948(v=office.14).aspx" >}}
- {{< reference
  title="Deploy Office Web Apps (Installed on SharePoint 2010 Products)"
  linkHref="http://technet.microsoft.com/en-us/library/ff431687(v=office.14).aspx" >}}

## Document conventions

To help you locate and interpret information easily, this document uses the
following style conventions and terminology.

{{< include-html "resources/table-1.html" >}}

# Overview of the installation process

The process for installing the Fabrikam Extranet is divided into the following
high-level steps.

## Step 1: Installation prerequisites

Before starting the installation:

- Various settings and configuration parameters need to be thoroughly planned
  out
- The environments and naming conventions need to be understood

## Step 2: Deploy and configure the server infrastructure

Prior to installing SharePoint Server 2010, the following tasks must be
completed:

- Windows Server 2008 must be installed and the server(s) must be joined to the
  domain
- Service accounts need to be created
- Any host names mapped to the loopback address need to be configured
- The Active Directory container used to track SharePoint 2010 installations
  should be created
- SQL Server 2008 must be installed and configured
- For the Development integration environment (DEV) and local development VMs,
  various development tools need to be installed (for example, Visual Studio 2010)

## Step 3: Install and configure SharePoint Server 2010

The installation and configuration of SharePoint Server 2010 consists of the
following tasks:

- Installing the prerequisite components on each SharePoint server that will
  participate in the farm
- Creating the SharePoint farm
- Joining additional servers to the farm
- Configuring diagnostic logging and usage and health data logging
- Configuring service accounts in SharePoint
- Configuring e-mail services in SharePoint
- Installing updates for SharePoint Server 2010

## Step 4: Create and configure the Web application

Creating and configuring the Web application consists of the following tasks:

- Setting environment variables used by various installation scripts
- Copying the build to the SharePoint server
- Creating the Web application and initial site collections
- Expanding content database files
- Configure the object cache user accounts (e.g. “Portal Super User”)
- Configuring the People Picker to support searches across the trust
  relationship between the extranet domain and Fabrikam's internal domain
- Configuring SSL on the Internet zone
- Enabling anonymous access to the top-level site
- Configuring claims-based authentication
- Enabling disk-based caching for the Web application

## Step 5: Configure service applications

In this step, various service applications – including the State Service and the
Search Service Application – are configured.

## Step 6: Install and configure Office Web Apps

Installing and configuring Office Web Apps consists of the following tasks:

- Installing Office Web Apps and registering the services in SharePoint
- Starting the service instances and creating the service applications
- Configuring the Excel Services Application trusted location
- Configuring the Office Web Apps cache
- Granting access to the content database

## Step 7: Deploy the Fabrikam solution

Deploying the Fabrikam solution consists of the following tasks:

- Configuring logging
- Installing the custom SharePoint solutions and activating the features
- Creating and configuring partner sites
- Creating and configuring team collaboration sites

# Installation prerequisites

## Plan the installation

There are a number of configuration parameters that need to be determined before
installing a solution based on SharePoint Server 2010. The various parameters
and configuration settings for the Fabrikam Extranet can be found in
[Appendix A - Planning worksheets](#Appendix-A-Planning-worksheets).

## Environments and naming conventions

Note that this guide typically specifies the values for the Production
environment. However it is also used to install and configure the Development
and Test environments, as well as local environments for developers.

A simple naming convention is recommended to differentiate between the various
environments.

Monikers – such as host headers and service accounts – in the Development
environment should utilize a “-dev” suffix. For example,
[http://extranet-dev.fabrikam.com](http://extranet-dev.fabrikam.com) is the URL
in the Development environment corresponding to
[http://extranet.fabrikam.com](http://extranet.fabrikam.com) in Production.
Similarly, FABRIKAM\svc-sharepoint-dev is the service account used in the
Development environment corresponding to EXTRANET\svc-sharepoint in Production.

{{< div-block "note" >}}

> **Note**
>
> Fabrikam has established an "extranet" Active Directory domain which is used
> to host the SharePoint farms for the Test and Production environments. In
> order to allow Fabrikam employees to authenticate with their internal domain
> (FABRIKAM) credentials, a one-way trust was configured from the EXTRANET
> domain to the FABRIKAM domain. To simplify the setup of the Development
> integration environment and local development VMs, the development servers are
> joined to the FABRIKAM domain.

{{< /div-block >}}

In order to distinguish sites on local development VMs, the “-local” suffix is
used. For example,
[http://extranet-local.fabrikam.com](http://extranet-local.fabrikam.com) is the
recommended URL on an individual developer’s VM corresponding to
[http://extranet.fabrikam.com](http://extranet.fabrikam.com) in Production. The
Hosts file (%WINDIR%\System32\Drivers\etc\hosts) is used to associate these URLs
with the loopback address (127.0.0.1).

Similarly, monikers in the Test environment should utilize a “-test” suffix. For
example, [http://extranet-test.fabrikam.com](http://extranet-test.fabrikam.com)
is the recommended URL in the Test environment corresponding to
[http://extranet.fabrikam.com](http://extranet.fabrikam.com) in Production.
Since the Test and Production environments share the same Active Directory
infrastructure, EXTRANET\svc-sharepoint-test is the service account used in the
Test environment corresponding to EXTRANET\svc-sharepoint in Production.

# Deploy and configure the server infrastructure

## Install Windows Server 2008

You must properly install Windows Server 2008 R2 (or the 64-bit edition of
Windows Server 2008) before installing SharePoint Server 2010. Install Windows
Server 2008 according to Fabrikam corporate standards. Ensure that all standard
programs, such as antivirus and server monitoring utilities, are installed and
configured.

### Use a Sysprep'ed image that includes the latest service pack and patches

Whenever possible, it is strongly recommended to start from a Sysprep'ed image
to save significant time. For information on creating and using Sysprep'ed
images for virtual machines, refer to the following blog posts:

- {{< reference title="Creating a VM/VHD Library"
  linkHref="/blog/jjameson/2010/04/02/creating-a-vm-vhd-library"
  linkText="https://www.technologytoolbox.com/blog/jjameson/2010/04/02/creating-a-vm-vhd-library" >}}
- {{< reference title="Using Sysprep'ed VHDs for New Hyper-V Virtual Machines"
  linkHref="/blog/jjameson/2009/08/13/using-sysprep-ed-vhds-for-new-hyper-v-virtual-machines"
  linkText="https://www.technologytoolbox.com/blog/jjameson/2009/08/13/using-sysprep-ed-vhds-for-new-hyper-v-virtual-machines" >}}

### Reset WSUS configuration

There is a known issue when using a Sysprep'ed image and Windows Server Update
Services (WSUS) to keep machines up-to-date with the latest patches.

To resolve the issue, remove the WSUS registry entries specified in
[KB 903262](http://support.microsoft.com/kb/903262):

1. Click **Start**, click **All Programs**, click **Accessories**, right-click
   **Command Prompt**, and then click **Run as administrator**.
1. At the command prompt, type the following commands:

   ```Batch
   net stop wuauserv

   reg delete HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate /v PingID /f
   reg delete HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate /v AccountDomainSid /f
   reg delete HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate /v SusClientId /f
   reg delete HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate /v SusClientIDValidation /f

   net start wuauserv
   ```

More information on this step is available in the following blog post:

{{< reference title="Script to Reset WSUS for Sysprep'ed Image" linkHref="/blog/jjameson/2011/04/25/script-to-reset-wsus-for-sysprep-ed-image" linkText="https://www.technologytoolbox.com/blog/jjameson/2011/04/25/script-to-reset-wsus-for-sysprep-ed-image" >}}

### Remove "stale" network adapters

When using Hyper-V and a Sysprep'ed image, a new network adapter is created the
first time the new VM is started (typically named "Microsoft Virtual Machine Bus
Network Adapter #2").

To cleanup the network adapters:

1. Click **Start**, click **All Programs**, click **Accessories**, right-click
   **Command Prompt**, and then click **Run as administrator**.
1. At the command prompt, type the following commands:

   ```Batch
   set devmgr_show_nonpresent_devices=1

   start devmgmt.msc
   ```

1. In the **Device Manager**window:
   1. Click the **View** menu and then click **Show hidden devices**.
   1. Expand **Network adapters**.
   1. Right-click each network adapter that begins with **Microsoft Virtual
      Machine Bus Network Adapter** (e.g. "Microsoft Virtual Machine Bus Network
      Adapter", "Microsoft Virtual Machine Bus Network Adapter #2") and click
      **Uninstall**. When prompted to confirm the device uninstall, click
      **OK**.
   1. If you notice any extra **Microsoft ISATAP Adapter** items, then uninstall
      those as well.
   1. Right-click **Network adapters** and then click **Scan for hardware
      changes**. (This will recreate the default adapter named "Microsoft
      Virtual Machine Bus Network Adapter".)

More information on this step is available in the following blog post:

{{< reference title="Removing \"Stale\" Network Adapters in Hyper-V VM" linkHref="/blog/jjameson/2011/03/14/removing-quot-stale-quot-network-adapters-in-hyper-v-vm" linkText="https://www.technologytoolbox.com/blog/jjameson/2011/03/14/removing-quot-stale-quot-network-adapters-in-hyper-v-vm" >}}

## DEV - Configure VM storage

Expand the primary VHD for development VMs to a minimum of 33 GB.

To improve the performance of SharePoint development environments, create two
additional virtual hard drives (D: and L:). Whenever possible, spread the VHD
files across multiple physical disks on the host, as illustrated in
[Figure 1](#Figure_1:_SharePoint_development_VM_configuration), to reduce I/O
contention.

{{< anchor name="Figure_1:_SharePoint_development_VM_configuration" >}}

{{< figure src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/SharePoint-Development-VM-600x317.png" alt="Figure 1: SharePoint development VM configuration" class="screenshot" height="317" width="600" caption="Figure 1: SharePoint development VM configuration" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/SharePoint-Development-VM-793x419.png)

{{< div-block "note" >}}

> **Note**
>
> Even if the VM host currently has only one physical disk (e.g. a developer's
> laptop), it is still recommended to create multiple VHD files. This greatly
> simplifies the process of distributing the I/O in the future when another
> physical drive is available.

{{< /div-block >}}

Since the amount of content in development environments is expected to be fairly
small, consider creating "data" and "log" VHDs of 1 GB and 500 MB, respectively.
Note that the Hyper-V management console does not currently support creating a
VHD smaller than 1 GB, but it is possible to create smaller VHDs using
PowerShell.

To create a 500 MB VHD for the SQL Server log files:

1. On the Hyper-V host, start Windows PowerShell with administrator privileges.
1. From the Windows PowerShell command prompt, type the following commands (update the VM name and VHD path as necessary):

   ```PowerShell
   $vhdService = Get-WmiObject -Class "Msvm_ImageManagementService" `
       -namespace "root\virtualization"

   $vhdService.CreateDynamicVirtualHardDisk(
       "D:\NotBackedUp\VMs\foobar5\foobar5_Log01.vhd",
       500MB)
   ```

More information on this step is available in the following blog post:

{{< reference title="Creating Small VHDs (< 1GB) for Hyper-V" linkHref="/blog/jjameson/2011/03/19/creating-small-vhds-lt-1gb-for-hyper-v" linkText="https://www.technologytoolbox.com/blog/jjameson/2011/03/19/creating-small-vhds-lt-1gb-for-hyper-v" >}}

## Set MaxPatchCacheSize to 0 (Optional)

To save significant disk space on development VMs, set the MaxPatchCacheSize
policy to 0 in the registry.

To configure the MaxPatchCacheSize policy:

1. Click **Start**, click **All Programs**, click **Accessories**, right-click
   **Command Prompt**, and then click **Run as administrator**.
1. At the command prompt, type the following command:

   {{< console-block >}}
   reg add HKLM\Software\Policies\Microsoft\Windows\Installer /v MaxPatchCacheSize /t REG_DWORD /d 0 /f
   {{< /console-block >}}

More information on this step is available in the following blog post:

{{< reference title="Save Significant Disk Space by Setting MaxPatchCacheSize to 0" linkHref="/blog/jjameson/2010/04/30/save-significant-disk-space-by-setting-maxpatchcachesize-to-0" linkText="https://www.technologytoolbox.com/blog/jjameson/2010/04/30/save-significant-disk-space-by-setting-maxpatchcachesize-to-0" >}}

## Disable Internet Protocol version 6 (TCP/IPv6)

To avoid potential issues with IPv6 addresses, disable TCP/IPv6 on the network
adapter(s).

To disable TCP/IPv6 on the network adapter(s):

1. In the **Network and Sharing Center**, click **Change adapter settings**.
1. In the **Network Connections** window, right-click the network adapter and
   then click **Properties**.
1. In the properties window for the network adapter, clear the checkbox for
   **Internet Protocol Version 6 (TCP/IPv6)** and then click **OK**.

## Join member server to domain

After completing the installation of Windows Server 2008, configure the server
to be a member of the extranet domain (EXTRANET).

{{< div-block "note" >}}

> **Note**
>
> Development environments do not have access to the extranet domain and
> therefore need to be joined to a different domain, such as FABRIKAM.

{{< /div-block >}}

## Install latest service pack and updates

Use Windows Update to install the latest service pack and updates for Windows
Server 2008.

## Create service accounts

A number of different service accounts need to be created in Active Directory
before installing SharePoint Server 2010.

To create the necessary service accounts:

1. Start the **Active Directory Users and Computers** console.
1. Select the node in the domain tree to contain the service accounts.
1. On the **Action** menu, point to **New**, and then click **User**.
1. In the **New User**dialog:
   1. Enter the information from [Table 2](#Table_2_-_Service_accounts).
   1. Click **Next**.
   1. Clear the **User must change password at next logon** checkbox.
   1. Select the **User cannot change password** checkbox.
   1. Select the **Password never expires** checkbox.
   1. Click **Next**.
   1. Click **Finish**.
1. Repeat steps 3 and 4 to create the remaining service accounts listed in Table 2.
1. Close the **Active Directory Users and Computers** console.

## Create Active Directory container to track SharePoint 2010 installations

In order to track SharePoint 2010 installations, a marker (called a Service
Connection Point) is created in Active Directory. To use this marker, create the
container in Active Directory and set the permissions for the container before
installing any SharePoint 2010 products in the environment.

To create a service connection point container to track installations:

1. On the domain controller, click **Start**, point to **Administrative Tools**,
   and then click **ADSI Edit**.
1. On the **Action** menu, click **Connect to**, and connect to the domain that
   you want to use.
1. In the console tree, expand the connection, expand the domain name, and then
   click **CN=System**.
1. In the **Actions** pane, under the **CN=System** heading, click **More
   Actions**, click **New**, and then click **Object...**
1. In the **Create Object**dialog box:
   1. In the **Select a class** list, click **container** and then click
      **Next**.
   1. In the **Value** box, type **Microsoft SharePoint Products** as the
      container name, and then click **Next**.
   1. Click **Finish**.
1. Double-click **CN=System** to view the items below it.
1. Right click the new container (**CN=Microsoft SharePoint Products**), and
   then click **Properties**.
1. In the **CN=Microsoft SharePoint Products Properties** window, on the
   **Security** tab, click **Advanced**.
1. In the **Advanced Security Settings for Microsoft SharePoint Products**
   window, on the **Permissions** tab, in the **Permission entries** list, click
   **Authenticated Users**, and then click **Edit**.
1. In the **Permission Entry for Microsoft SharePoint Products** window, in the
   **Permissions** list, select the **Allow** checkbox for **Create
   serviceConnectionPoint objects**, and then click **OK**.
1. In the **Advanced Security Settings for Microsoft SharePoint Products**
   window, click **OK**.
1. In the **CN=Microsoft SharePoint Products Properties** window, click **OK**.

## DEV – Map Web application to loopback address in Hosts file

On local development VMs, the Hosts file is used to associate the URLs of
SharePoint Web applications with the loopback address (127.0.0.1).

To map the host name for a Web application to the loopback address:

1. Click **Start**, click **All Programs**, click **Accessories**, right-click
   **Command Prompt**, and then click **Run as administrator**.
1. At the command prompt, type the following command:

   ```Batch
   notepad %WINDIR%\System32\Drivers\etc\hosts
   ```

1. In Notepad, add a line to map the loopback address (127.0.0.1) to the "local" version of each host header specified in [Table 7](#Table_7_-_Web_applications). For example:

   ```Text
   127.0.0.1 	extranet-local.fabrikam.com
   ```

1. Save the changes to the file and close the editor.

## Allow specific host names mapped to 127.0.0.1

Windows Server 2008 includes a loopback check security feature that is designed
to help prevent reflection attacks. However this feature is problematic when
using host names mapped to 127.0.0.1 on local development machines.

{{< div-block "note" >}}

> **Note**
>
> This problem was originally encountered only in local development environments
> (where the host names are mapped to the loopback address). However, it was
> later encountered in other environments after patches were installed from
> Windows Update. Consequently, you may need to perform the following
> configuration steps if HTTP 401.1 errors occur when accessing the site locally
> on the server (regardless of environment).

{{< /div-block >}}

More details about this issue can be found in
[KB 896861](http://support.microsoft.com/kb/896861).

To enable host names that are mapped to the loopback address:

1. Click **Start**, click **Run**, type **regedit**, and then click **OK**. If
   prompted by User Account Control to allow the program to make changes to the
   computer, click **Yes**.
1. In Registry Editor, locate and then click the following registry key:

   > **HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa\MSV1_0**

1. Right-click **MSV1_0**, point to **New**, and then click **Multi-String
   Value**.
1. Type **BackConnectionHostNames**, and then press {{< kbd "ENTER" >}}.
1. Right-click **BackConnectionHostNames**, and then click **Modify**.
1. In the **Value data** box, type the host header corresponding to each Web application specified in [Table 7](#Table_7_-_Web_applications). For example:

   > {{< kbd "extranet.fabrikam.com" >}}
   >
   > **Note**
   >
   > Be sure to specify the host header corresponding to the environment. For
   > example, specify {{< kbd "extranet-local.fabrikam.com" >}} for local
   > development environments.

1. Quit Registry Editor, and then restart the IIS Admin Service.

## DEV - Install Windows PowerShell Integrated Scripting Environment

To install Windows PowerShell Integrated Scripting Environment (ISE):

1. Start the **Server Manager** console.
1. In the tree view on the left, click **Features**, and then in the **Features
   Summary** area, click **Add Features**.
1. In the **Add Features Wizard**:
   1. On the **Features**page:
      1. In the list of features, select **Windows PowerShell Integrated
         Scripting Environment (ISE)**. If prompted to add features required for
         Windows PowerShell Integrated Scripting Environment (ISE), click **Add
         Required Features**.
      1. Click **Next**.
   1. On the **Confirmation** page, verify the features to be installed and then
      click **Install**.

## DEV - Install Visual Studio 2010

To install Visual Studio 2010:

1. Start the Visual Studio 2010 installation.
1. On the **Welcome to the Microsoft Visual Studio 2010 installation wizard**,
   click **Next**.
1. Review the licensing agreement. If you accept the terms and conditions,
   select **I have read and accept the license terms**, and then click **Next**.
1. On the **Select features to install** step, select **Custom** and then click
   **Next**.
1. Clear the following checkboxes:
   - **Visual C++**
   - **Visual F#**
   - **Dotfuscator Software Services - Community Edition**
   - **Microsoft SQL Server 2008 Express Service Pack 1 (x64)**
1. Click **Install**.
1. The **Installing Components** page shows the progress of the installation.
1. Wait for the installation to complete, verify all components were
   successfully installed, and then click **Finish**.

## DEV - Install Team Explorer (Team Foundation Client)

{{< div-block "note" >}}

> **Note**
>
> Depending on which edition of Visual Studio 2010 was installed in the previous
> step, Team Explorer may already be installed.

{{< /div-block >}}

To install Team Explorer:

1. Open Windows Explorer, and browse to the installation media for Visual Studio 2010.
1. Open the **Team Explorer** folder, and double-click **setup.exe**.
1. The **Microsoft Visual Studio Team Explorer 2010 Setup** wizard starts.
1. On the **Welcome to Setup** page, click **Next**.
1. Review the licensing agreement. If you accept the terms and conditions,
   select **I have read and accept the license terms**, and then click **Next**.
1. On the **Select features to install** page, ensure **Team Explorer** is
   checked and then click **Install**.
1. The **Installing Components** page shows the progress of the installation.
1. Wait for the installation to complete, verify all components were
   successfully installed, and then click **Finish**.

## DEV - Install Visual Studio 2010 Service Pack 1

Install Visual Studio SP1 using Windows Update or by downloading and running the
installer from the following location:

{{< reference title="Microsoft Visual Studio 2010 Service Pack 1 (Installer)" linkHref="http://www.microsoft.com/download/en/details.aspx?displaylang=en&id=23691" >}}

## DEV – Install TFS Power Tools

In order to leverage additional check-in policies, install the TFS Power Tools
(using the default options):

{{< reference title="Visual Studio Power Tools" linkHref="http://msdn.microsoft.com/en-us/vstudio/bb980963.aspx" >}}

## Install SQL Server 2008

For development environments, use the following steps to install the 64-bit
edition of SQL Server 2008 R2 (or the 64-bit version of SQL Server 2008).

{{< div-block "note important" >}}

> **Important**
>
> For the Test and Production environments, refer to the SQL Server Installation
> Guide to install SQL Server on the dedicated server or cluster.

{{< /div-block >}}

SQL Server 2008 setup requires .NET Framework 3.5 to be installed.

{{< div-block "note" >}}

> **Note**
>
> SQL Server 2008 R2 checks for the .NET Framework dependency and can enable the
> .NET Framework Core role as part of the setup process. Also note that if
> Windows PowerShell Integrated Scripting Environment (ISE) was installed
> earlier then .NET Framework 3.5 has already been installed since it is a
> dependency of that feature.

{{< /div-block >}}

To install .NET Framework 3.5 on Windows Server 2008:

1. Start the **Server Manager** console.
1. In the tree view on the left, click **Features**, and then in the **Features
   Summary** area, click **Add Features**.
1. In the **Add Features Wizard**:
   1. On the **Features** page, in the list of features, expand **.NET Framework
      3.5.1 Features**, select **.NET Framework 3.5.1**, and then click
      **Next**.
   1. On the **Confirmation** page, verify the features to be installed and then
      click **Install**.
1. Run Windows Update to install the latest updates for the .NET Framework.
1. Restart the computer.

To install SQL Server:

1. Start the SQL Server installation. If prompted about known compatibility
   issues (indicating that SQL Server 2008 SP1 must be applied), click **Run
   program**.
1. On the **SQL Server Installation Center**, click **Installation**, and then
   click **New installation or add features to an existing installation**.
1. On the **Setup Support Rules** step, ensure no failures or warnings were
   detected and then click **OK**.
1. On the **Product Key** step, type the product key, if necessary, and then
   click **Next**.
1. Review the licensing agreement. If you accept the terms and conditions,
   select **I accept the license terms**, and then click **Next**.
1. On the **Setup Support Files** step, click **Install**. Wait for the support
   files to be installed.
1. On the **Setup Support Rules** step, ensure no failures or warnings were
   detected (except for a warning regarding **Windows Firewall**) and then click
   **Next**.
1. On the **Setup Role** step, ensure **SQL Server Feature Installation** is
   selected and then click **Next**.
1. On the **Feature Selection**step:
   1. Select the following checkboxes:
      - **Database Engine Services**
      - **SQL Server Books Online**
      - **Management Tools - Complete**
   1. Click **Next**.
1. On the **Installation Rules** step, ensure no failures or warnings were
   detected and then click **Next**.
1. On the **Instance Configuration** step, ensure **Default instance** is
   selected, the **Instance ID** is set to **MSSQLSERVER**, and then click
   **Next**.
1. On the **Disk Space Requirements** page, ensure there is sufficient disk
   space available, and then click **Next**.
1. On the **Server Configuration** step, on the **Service Accounts**tab:
   1. Enter the service accounts as specified in
      [Table 2](#Table_2_-_Service_accounts).
   1. Ensure the **Startup Type** for **SQL Server Database Engine** is set to
      **Automatic**.
   1. Click **Next.**
1. On the **Database Engine Configuration**step:
   1. On the **Account Provisioning**tab:
      1. Ensure that **Windows authentication mode** is selected.
      1. In the **Specify SQL Server administrators** section, click **Add
         Current User**.
   1. On the **Data Directories** tab, specify the desired location of the
      various data and log files (for example, to place the data files on D: and
      the log files on L:).
   1. Click **Next**.
1. On the **Error Reporting** step, optionally select the checkbox to send error
   reports to Microsoft or your corporate report server, and then click
   **Next**.
1. On the **Installation Configuration Rules** step, ensure no failures or
   warnings were detected and then click **Next**.
1. On the **Ready to Install** step, verify the components that will be
   installed and then click **Install**.
1. Wait for the installation to complete, verify all components were
   successfully installed, and then click **Next**.
1. On the **Complete** step, click **Close**.

## Install latest service pack for SQL Server 2008

Install SQL Server 2008 R2 Service Pack 2 or SQL Server 2008 Service Pack 3
using Windows Update or by downloading it from one the following locations:

- {{< reference title="Microsoft® SQL Server® 2008 R2 Service Pack 2"
  linkHref="http://www.microsoft.com/en-us/download/details.aspx?id=30437" >}}
- {{< reference title="SQL Server 2008 Service Pack 3"
  linkHref="http://www.microsoft.com/download/en/details.aspx?displaylang=en&id=27594" >}}

## DEV - Change databases to Simple recovery model

Using the Simple recovery model in SQL Server alleviates the need to
periodically backup transaction logs in development environments and also allows
a very small VHD to be used for the log files.

Execute the following SQL script to change all user databases and the
out-of-the-box **model** database to use the Simple recovery model:

```SQL
IF OBJECT_ID('tempdb..#CommandQueue') IS NOT NULL DROP TABLE #CommandQueue

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

More information on this step is available in the following blog post:

{{< reference title="Using the Simple Recovery Model for SharePoint Development Environments" linkHref="/blog/jjameson/2011/03/19/using-the-simple-recovery-model-for-sharepoint-development-environments" linkText="https://www.technologytoolbox.com/blog/jjameson/2011/03/19/using-the-simple-recovery-model-for-sharepoint-development-environments" >}}

## DEV - Install Microsoft Office 2010

Install the 32-bit edition of Microsoft Office 2010 (using the default
installation options).

## DEV - Install Microsoft SharePoint Designer 2010

Install the 32-bit edition of Microsoft SharePoint Designer 2010 (using the
default installation options).

## DEV - Install Microsoft Visio 2010

Install the 32-bit edition of Microsoft Visio 2010 (using the default
installation options).

## DEV - Install additional service packs and updates

Install the latest service packs and patches for Microsoft Office and other
products using Windows Update.

## DEV - Install additional browsers and Adobe Reader

For development and testing purposes, install Mozilla Firefox, Google Chrome,
and Adobe Reader.

# Install and configure SharePoint Server 2010

## Prepare the farm servers

Before installing SharePoint Server, you must install the prerequisites on the
SharePoint application and Web servers by using the Microsoft SharePoint
Products Preparation Tool.

Use the following procedure to install prerequisites on each of the farm
servers.

To run the preparation tool:

1. From the SharePoint Server 2010 installation location, double-click the
   appropriate executable file.
1. Click **Install software prerequisites** on the splash screen. If prompted by
   User Account Control to allow the program to make changes to the computer,
   click **Yes**.
1. On the **Welcome to the Microsoft® SharePoint® 2010 Products Preparation
   Tool** page, click **Next**.
1. Review the licensing agreement. If you accept the terms and conditions, select **I accept the terms of the License Agreement(s)**, and then click **Next**.

   {{< div-block "note" >}}

   > **Note**
   >
   > The preparation tool may have to restart the server to complete the
   > installation of some of the prerequisites. The installer will continue to
   > run after the server is restarted, and no manual intervention is required.
   > However, you will have to log back on to the server.

   {{< /div-block >}}

1. On the **Installation Complete** page, click **Finish**.
1. Restart the server to complete the installation of the prerequisites.

   {{< div-block "note" >}}

   > **Note**
   >
   > After you complete the Microsoft SharePoint Products Preparation Tool, you
   > must install [KB 949516](http://go.microsoft.com/fwlink/?LinkId=148917) and
   > [KB 971831](http://go.microsoft.com/fwlink/?LinkID=165750). Restart the
   > server after installing these hotfixes.
   >
   > When installing these hotfixes on Windows Server 2008 R2, you may be
   > notified that these updates are not applicable.

   {{< /div-block >}}

   {{< div-block "note" >}}

   > **Note**
   >
   > If the error message <q class="directQuote errorMessage">"Loading this
   > assembly would produce a different grant set from other instances.
   > (Exception from HRESULT: 0x80131401)"</q> is displayed when you start the
   > IIS worker process (w3wp.exe), another service, or a managed application on
   > a server that is also running SharePoint Server 2010, you must install
   > [KB 963676](http://go.microsoft.com/fwlink/?LinkId=151358). You must
   > restart the computer after you apply this hotfix.

   {{< /div-block >}}

## Install security update for Web applications using claims authentication

Web applications that use claims-based authentication are at risk for a
potential security vulnerability that might allow users to elevate privileges.
To resolve this issue, a security update is required on each SharePoint server
in the farm.

Download and install the update from the following location:

{{< reference title="KB979917 - QFE for Sharepoint issues - Perf Counter fix & User Impersonation" linkHref="http://code.msdn.microsoft.com/KB979917" >}}

{{< div-block "note important" >}}

> **Important**
>
> This hotfix is included in Windows Server 2008 R2 Service Pack 1.

{{< /div-block >}}

{{< div-block "note" >}}

> **Note**
>
> In the list of files to download, there is no option corresponding to Windows
> Server 2008 (only Windows Vista and Windows 7). For Windows Server 2008 or
> Windows Server 2008 R2, download and install the Windows 7 x64 version:
>
> > **Windows6.1-KB979917-x64.msu (Win7)**

{{< /div-block >}}

Be sure to install the update on each SharePoint server in the farm.

## Install SharePoint Server 2010 on the farm servers

After the prerequisites are installed, use the following procedure to install
SharePoint Server on each of the farm servers.

To install SharePoint Server 2010:

1. If the SharePoint installation splash screen is not already showing, from the
   SharePoint Server 2010 installation location, double-click the appropriate
   executable file.
1. Click **Install SharePoint Server** on the splash screen. If prompted by User
   Account Control to allow the program to make changes to the computer, click
   **Yes**.
1. On the **Enter your Product Key** page, type the corresponding SharePoint
   Server 2010 Enterprise CAL product key, and then click **Continue**.
1. Review the licensing agreement. If you accept the terms and conditions,
   select **I accept the terms of this agreement**, and then click **Continue**.
1. On the **Choose the installation you want** page, click **Server Farm**.
1. On the **Server Type** tab, click **Complete**.
1. On the **File Location** tab, change the installation and search index paths
   according to [Table 3](#Table_3_-_Installation_file_locations), and then
   click **Install Now**.
1. Wait for the installation to finish.
1. On the **Run Configuration Wizard** page, clear the **Run the SharePoint Products and Technologies Configuration Wizard now** checkbox, and then click **Close**.

   {{< div-block "note" >}}

   > **Note**
   >
   > For consistency of approach, it is recommended that you do not run the
   > configuration wizard until SharePoint Server has been installed on all
   > application and front-end Web servers that will participate in the server
   > farm.

   {{< /div-block >}}

## Create and configure the farm

To create and configure the farm, you run the SharePoint Products Configuration
Wizard. This wizard automates several configuration tasks, including creating
the configuration database, installing services, and creating the Central
Administration Web site.

{{< div-block "note important" >}}

> **Important**
>
> Run the SharePoint Products Configuration Wizard on the server that will host
> the Central Administration Web site before you run the wizard on the other
> servers in the farm.

{{< /div-block >}}

To run the configuration wizard and configure the farm:

1. On the server that will host the Central Administration site, click
   **Start**, point to **All Programs**, click **Microsoft SharePoint 2010
   Products**, and then click **SharePoint 2010 Products Configuration Wizard**.
   If prompted by User Account Control to allow the program to make changes to
   the computer, click **Yes**.
1. On the **Welcome to SharePoint Products** page, click **Next**.
1. In the dialog box that notifies you that some services may have to be started
   or reset during configuration, click **Yes**.
1. On the **Connect to a server farm** page, click **Create a new server farm**,
   and then click **Next**.
1. On **the Specify Configuration Database Settings**page, do the following:
   1. In the **Database server** box, type the name of the computer that is
      running SQL Server.
   1. In the **Username** box, type the user name of the server farm account
      specified in [Table 2](#Table_2_-_Service_accounts).
   1. In the **Password** box, type the password for the service account.
   1. Click **Next**.
1. On the **Specify Farm Security Settings** page, type a passphrase, and then click **Next**.
   Ensure that the passphrase meets the following criteria:

   - Contains at least eight characters
   - Contains at least three of the following four character groups:
     - English uppercase characters (from A through Z)
     - English lowercase characters (from a through z)
     - Numerals (from 0 through 9)
     - Nonalphabetic characters (such as !, $, #, %)

   {{< div-block "note" >}}

   > **Note**
   >
   > Although a passphrase is similar to a password, it is usually longer to
   > enhance security. It is used to encrypt credentials of accounts that are
   > registered in SharePoint Server 2010. For example, the SharePoint Server
   > 2010 system account that you provide when you run the SharePoint Products
   > Configuration Wizard wizard. Ensure that you remember the passphrase,
   > because you must use it each time you add a server to the farm.

   {{< /div-block >}}

1. On the **Configure SharePoint Central Administration Web Application**page,
   do the following:
   1. Select the **Specify port number** checkbox and type the port number
      specified in [Table 7](#Table_7_-_Web_applications).
   1. In the **Configure Security Settings** section, ensure NTLM is selected.
   1. Click **Next**.
1. On the **Completing the SharePoint Products Configuration Wizard** page,
   verify the configuration settings, and then click **Next**.
1. Wait for the product configuration to complete.
1. On the **Configuration Successful** page, click **Finish**.
1. The Central Administration Web site will open in a new browser window.

   {{< div-block "note" >}}

   > **Note**
   >
   > If you are prompted for your user name and password, you need to add the
   > SharePoint Central Administration site to the **Local intranet** zone and
   > configure the default settings for this zone in Internet Explorer.
   > Instructions for configuring these settings are provided in a following
   > section
   > ([Add SharePoint Central Administration to the Local intranet zone](#Add_SharePoint_Central_Administration_to_the_Local_intranet_zone)).
   > In order to complete the steps in this section, type your username and
   > password to access Central Administration.

   {{< /div-block >}}

1. In the **Help Make SharePoint Better** window, click the desired option and
   then click **OK**.
1. On the **Configure your SharePoint farm** page, click **Cancel** (since the
   services will be configured manually).

## Add Web servers to the farm

After creating the farm on the first server, add the remaining servers by
following the process described in the previous section. However, rather than
selecting the option to create a new farm, instead select the option to join an
existing farm and then follow the wizard steps to join the farm.

## Add SharePoint Central Administration to the Local intranet zone {#Add_SharePoint_Central_Administration_to_the_Local_intranet_zone}

Adding the Central Administration site to the **Local intranet** zone (and using
the default settings for this zone) enables single sign-on when accessing the
site. This task should be completed on the server that hosts the Central
Administration site. It can also be performed on other servers used to access
the Central Administration site.

To add the Central Administration site to the Local intranet zone:

1. In Internet Explorer, on the **Tools** menu, click **Internet Options**.
1. On the **Security** tab, in the **Select a zone to view or change security
   settings** box, click **Local intranet**, and then click **Sites**.
1. Clear the **Require server verification (https:) for all sites in this zone**
   checkbox.
1. In the **Add this Web site to the zone** box, type the URL for the SharePoint
   Central Administration Web site, and then click **Add**.
1. Click **Close** to close the **Local intranet** dialog box.
1. In the **Security level for this zone** section, click **Default level**.
1. Click **OK** to close the **Internet Options** dialog box.

More information on this step is available in the following blog post:

{{< reference title="Be \"In the Zone\" to Avoid Entering Credentials" linkHref="/blog/jjameson/2007/03/22/be-in-the-zone-to-avoid-entering-credentials" linkText="https://www.technologytoolbox.com/blog/jjameson/2007/03/22/be-in-the-zone-to-avoid-entering-credentials" >}}

## Add the URL for the extranet website to the Local intranet zone

In order to access the extranet website locally on the SharePoint server, follow
the steps in the previous section to add
[http://extranet.fabrikam.com](http://extranet.fabrikam.com) to the **Local
intranet** zone.

## Add the SharePoint bin folder to the PATH environment variable

On each SharePoint server in the farm, append **C:\Program Files\Common
Files\Microsoft Shared\web server extensions\14\BIN** to the **PATH**
environment variable (in order to run stsadm.exe from various folder locations
without having to specify the full path).

## Grant DCOM permissions on IIS WAMREG admin Service

In order to avoid errors in the Windows event log (e.g. Event ID 10016), grant
the WSS_ADMIN_WPG and WSS_WPG groups appropriate permissions on the IIS WAMREG
admin Service, as described in
[KB 920783](http://support.microsoft.com/kb/920783).

{{< div-block "note important" >}}

> **Important**
>
> This task must be completed on each SharePoint server in the farm.

{{< /div-block >}}

{{< div-block "note" >}}

> **Note**
>
> If performing this step on Windows Server 2008 R2, you must first take
> ownership of the corresponding registry key and grant administrators
> permissions to update the configuration.

{{< /div-block >}}

To grant Administrators permissions to update the configuration in Windows
Server 2008 R2:

1. Click the **Start** menu, type **regedit**, and then click **regedit.exe**.
   If prompted by **User Account Control** to allow the program to make changes
   to this computer, click **Yes**.
1. In the **Registry Editor** window, search for
   "61738644-F196-11D0-9953-00C04FD919C1" to find:

   > HKEY_CLASSES_ROOT\AppID\\{61738644-F196-11D0-9953-00C04FD919C1}

1. Right-click on the
   **HKEY_CLASSES_ROOT\AppID\\{61738644-F196-11D0-9953-00C04FD919C1}** key and
   then click **Permissions**.
1. In the **Permissions for {61738644-F196-11D0-9953-00C04FD919C1}** dialog box,
   click **Advanced**.
1. In the **Advanced Security Settings for
   {61738644-F196-11D0-9953-00C04FD919C1}**dialog box:
   1. Click the **Owner** tab.
   1. In the **Change owner to** list, click the **Administrators** group.
   1. Click **OK**.
1. In the **Permissions for {61738644-F196-11D0-9953-00C04FD919C1}** dialog box,
   click the **Administrators** group, then click the checkbox to allow the
   group **Full Control**, and click **OK**.
1. Close the Registry Editor window.

To configure permissions for the IIS WAMREG Admin Service:

1. Click the **Start** menu, type **dcomcnfg**, and then click **dcomcnfg.exe**.
1. Expand **Component Services**, expand **Computers**, expand **My Computer**,
   and then click **DCOM Config**.
1. Right-click **IIS WAMREG admin Service**, and then click **Properties**.
1. Click the **Security** tab.
1. Under **Launch and Activation Permissions**, click **Edit**.
1. In the **Launch and Activation Permission** dialog box, click **Add**.
1. In the **Select Users, Computers, Service Accounts, or Groups** dialog box,
   change the location to the local server. Then, type the local security groups
   **WSS_ADMIN_WPG** and **WSS_WPG**, click **Check Names**, and then click
   **OK**.
1. In the **Launch and Activation Permission**dialog box:
   1. In the **Groups or user names** list, click **WSS_ADMIN_WPG**, in the
      **Permissions for WSS_ADMIN_WPG** list, click to select the **Allow**
      checkbox that is next to **Local Activation**.
   1. Repeat the previous step to grant the **Local Activation** permission for
      the **WSS_WPG** group.
   1. Click **OK**.
1. In the **IIS WAMREG admin Service Properties** dialog box, click **OK**.

More information on this step is available in the following blog post:

{{< reference title="Event ID 10016, KB 920783, and the WSS_WPG Group" linkHref="/blog/jjameson/2009/10/17/event-id-10016-kb-920783-and-the-wss-wpg-group" linkText="https://www.technologytoolbox.com/blog/jjameson/2009/10/17/event-id-10016-kb-920783-and-the-wss-wpg-group" >}}

## Rename TaxonomyPicker.ascx

In order to avoid errors in the Windows event log (e.g. Source: SharePoint
Foundation, Event ID: 7043), rename the out-of-the-box TaxonomyPicker.ascx file.

{{< div-block "note important" >}}

> **Important**
>
> This task must be completed on each SharePoint server in the farm.

{{< /div-block >}}

To rename the TaxonomyPicker.ascx file:

1. Open Windows Explorer and browse to the following folder:

   > **C:\Program Files\Common Files\Microsoft Shared\Web Server
   > Extensions\14\TEMPLATE\CONTROLTEMPLATES**

1. Right-click **TaxonomyPicker.ascx**, click **Rename**, and then change the
   filename to **TaxonomyPicker.ascx_broken**. When prompted to confirm that you
   want to change the file name extension, click **Yes**.

{{< div-block "note" >}}

> **Note**
>
> Changing the file extension causes the problematic file to be skipped by
> ASP.NET when compiling the controls in the folder.

{{< /div-block >}}

## Configure diagnostic logging and usage and health data collection

After you add the remaining servers to the farm, configure initial diagnostic
logging and usage and health data collection for the farm.

To configure diagnostic logging:

1. Click **Start**, point to **All Programs**, click **Microsoft SharePoint 2010
   Products**, and then click **SharePoint 2010 Central Administration**. If
   prompted by User Account Control to allow the program to make changes to the
   computer, click **Yes**.
1. On the Central Administration home page, click **Monitoring**.
1. In the **Reporting** section, click **Configure diagnostic logging**.
1. On the **Diagnostic Logging** page, configure the settings as specified in
   [Table 4](#Table_4_-_Diagnostic_logging), and then click **OK**.

To configure usage and health data collection:

1. On the **Monitoring** page in Central Administration, in the **Reporting**
   section, click **Configure usage and health data collection**.
1. On the **Configure web analytics and health data collection** page, configure
   the settings as specified in
   [Table 5](#Table_5_-_Web_analytics_and_health_data_collection), and then
   click **OK**.

## Configure service accounts

Managed accounts are used for various farm components in SharePoint 2010. In
order to specify a service account when configuring various SharePoint features,
the service account must first be registered as a managed account in Central
Administration.

{{< div-block "note" >}}

> **Note**
>
> If using the installation scripts, instead of Central Administration, in later
> sections of this document to create and configure the Web application and
> other service applications, then you may skip this step and instead specify
> the credentials when running the scripts.

{{< /div-block >}}

To configure service accounts:

1. On the Central Administration home page, under the **Security** section,
   click **Configure service accounts**.
1. On the **Service Accounts** page, click **Register new managed account**.
1. On the **Register Managed Account** page, in the **Account Registration**
   section, type the username and password for the service account for search
   services listed in [Table 2](#Table_2_-_Service_accounts), and the click
   **OK**.
1. Repeat the previous steps to add the remaining SharePoint managed accounts
   listed in Table 2.

## Configure mail services

This section describes how to configure outgoing e-mail in order to send e-mail
alerts to site users and notifications to site administrators.

To configure the outgoing e-mail settings:

1. On the Central Administration home page, click **System Settings**.
1. On the **System Settings** page, in the **E-Mail and Text Messages (SMS)**
   section, click **Configure outgoing e-mail settings**.
1. On the **Outgoing E-Mail Settings** page, in the **Mail Settings** section,
   configure the following settings from the values specified in
   [Table 6](#Table_6_-_Outgoing_e-mail_settings):
   - **Outbound SMTP server**
   - **From address**
   - **Reply-to address**
   - **Character set**
1. Click **OK**.

## DEV - Configure timer job history

SharePoint Server 2010 is configured by default to preserve 7 days of historical
information about timer jobs. However, the cleanup job is scheduled to run once
per week on Sundays. For development environments that are powered off on
weekends, this cleanup job may never run and consequently the growth of the job
history table in the SharePoint_Config database can cause issues.

To change the schedule for deleting timer job history:

1. On the **Start** menu, click **All Programs**, click **Microsoft SharePoint
   2010 Products**, right-click **SharePoint 2010 Management Shell**, and then
   click **Run as administrator**. If prompted by User Account Control to allow
   the program to make changes to the computer, click **Yes**.
1. From the Windows PowerShell command prompt, type the following command:

   ```PowerShell
   Set-SPTimerJob "job-delete-job-history" -Schedule "Daily between 12:00:00 and 13:00:00"
   ```

1. Wait for the command to complete and verify no errors occurred during the
   process.

## Install SharePoint Server 2010 Service Pack 1

Install SharePoint Server 2010 Service Pack 1 on each server in the farm:

{{< reference title="Service Pack 1 for Microsoft SharePoint Server 2010 (KB2460045)" linkHref="http://www.microsoft.com/en-us/download/details.aspx?id=26623" >}}

{{< div-block "note" >}}

> **Note**
>
> To save time, it is recommended that you do not run the configuration wizard
> after installing the service pack. Wait until the cumulative update is
> installed (as described in the next section) to complete the farm upgrade.

{{< /div-block >}}

## Install August 2012 Cumulative Update

Install the SharePoint Server 2010 cumulative update for August 2012:

{{< reference title="Description of the SharePoint Server 2010 cumulative update package (SharePoint server-package): August 28, 2012" linkHref="http://support.microsoft.com/kb/2687353" >}}

After installing the update on each server in the farm, run the SharePoint
Products and Technologies Configuration Wizard (PSConfigUI.exe) to upgrade the
farm.

# Create and configure the Web application

## Set environment variables

By default, the installation scripts for the Fabrikam Extranet solution install
Release builds to [http://extranet.fabrikam.com](http://extranet.fabrikam.com).
If installing the solution to a different URL, set the environment variable
FABRIKAM_EXTRANET_URL to the URL of the site. To install Debug builds, set the
environment variable FABRIKAM_BUILD_CONFIGURATION to Debug.

For example, [Figure 2](#Figure_2:_Setting_environment_variables) shows the
environment variables for a local development VM (where
FABRIKAM_BUILD_CONFIGURATION = Debug and FABRIKAM_EXTRANET_URL =
[http://extranet-local.fabrikam.com](http://extranet-local.fabrikam.com)).

{{< anchor name="Figure_2:_Setting_environment_variables" >}}

{{< figure src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Environment-Variables-Fabrikam-Extranet-394x436.png" alt="Setting environment variables" class="screenshot" height="436" width="394" caption="Figure 2: Setting environment variables" >}}

## DEV - Snapshot VM

To allow the ability to quickly rollback the development environment to a
"clean" state, create a snapshot of the virtual machine at this point. Name the
snapshot **Baseline SharePoint Server 2010 configuration**.

More information about using VM snapshots for SharePoint development is
available in the following blog posts:

- {{< reference
  title="Virtual Machine Snapshots and SharePoint Development, Part 1"
  linkHref="/blog/jjameson/2011/03/22/virtual-machine-snapshots-and-sharepoint-development-part-1"
  linkText="https://www.technologytoolbox.com/blog/jjameson/2011/03/22/virtual-machine-snapshots-and-sharepoint-development-part-1" >}}
- {{< reference
  title="Virtual Machine Snapshots and SharePoint Development, Part 2"
  linkHref="/blog/jjameson/2011/03/23/virtual-machine-snapshots-and-sharepoint-development-part-2"
  linkText="https://www.technologytoolbox.com/blog/jjameson/2011/03/23/virtual-machine-snapshots-and-sharepoint-development-part-2" >}}

## Copy Fabrikam Extranet build to SharePoint server

Copy the Fabrikam Extranet build from the release server (e.g.
[\\\\DAZZLER\Builds\Fabrikam\Demo\SharePointExtranet\\{build version}](file://DAZZLER/Builds/Fabrikam/Demo))
to the SharePoint server running Central Administration.

For the Test and Production environments, always designate the specific build to
be deployed. For example:

{{< console-block >}}

robocopy \\\\DAZZLER\Builds\Fabrikam\Demo\SharePointExtranet\1.0.176.0
C:\NotBackedUp\Fabrikam\Demo\SharePointExtranet\1.0.176.0 /E /MIR

{{< /console-block >}}

For development environments, the "latest" version may be specified. For
example:

{{< console-block >}}

robocopy \\\\DAZZLER\Builds\Fabrikam\Demo\SharePointExtranet\\\_latest
C:\NotBackedUp\Fabrikam\Demo\SharePointExtranet\\\_latest /E /MIR

{{< /console-block >}}

Developers may alternately choose to deploy to local environments directly from
a TFS workspace for a specific branch (e.g. C:\NotBackedUp\Fabrikam\Demo\Main).

## Create the Web application and initial site collections

{{< div-block "note important" >}}

> **Important**
>
> Prior to creating the Fabrikam Extranet Web application (and the associated
> content database), it is strongly recommended that SQL Server first be
> configured to create database files and transaction log files in the desired
> locations. This eliminates the need to move the data and/or log files after
> the databases have been created.

{{< /div-block >}}

The Web application and initial site collections can either be created using
PowerShell scripts or through Central Administration. Using the PowerShell
scripts will result in significantly shorter deployment times, whereas
completing the equivalent procedures through Central Administration will provide
greater familiarity with the various configuration options in SharePoint Server 2010.

To create the Web application using the PowerShell scripts:

1. On the **Start** menu, click **All Programs**, click Microsoft SharePoint
   2010 Products, right-click **SharePoint 2010 Management Shell**, and then
   click **Run as administrator**. If prompted by User Account Control to allow
   the program to make changes to the computer, click **Yes**.
1. From the Windows PowerShell command prompt, change to the following
   directory:

   > **{build version or branch folder}\\[Source]\Deployment Files\Scripts**

1. Type the following command:

   ```PowerShell
   & '.\Create Web Application.ps1'
   ```

1. If prompted for the the application pool credentials, verify the user name,
   type the password, and then click **OK**.
1. Wait for the script to complete and verify no errors occurred during the
   process.
1. Type the following command:

   ```PowerShell
   & '.\Create Site Collections.ps1'
   ```

1. Wait for the script to complete and verify no errors occurred during the
   process.
1. Proceed to the next section
   ([Expand content database files](#Expand_content_database_files)).

To create the Web application using SharePoint Central Administration:

1. On the Central Administration home page, in the **Application Management**
   section, click **Manage web applications**.
1. On the **Web Applications** tab, in the ribbon, click **New**.
1. On the **Create New Web Application**page:

   1. In the **Authentication** section, click **Claims Based Authentication**.
   1. In the **IIS Web Site**section:
      1. Click **Create a new IIS web site**.
      1. In the **Port** and **Host Header** boxes, type the corresponding
         values for the **Fabrikam Extranet** Web application specified in
         [Table 7.](#Table_7_-_Web_applications)
   1. In the **Security Configuration** section, under **Allow Anonymous**,
      click **Yes**.
   1. In the **Claims Authentication Types**section:

      <div class="overflow-auto">

      1. Ensure the **Enable Windows Authentication** checkbox is selected and
         in the **Integrated Windows authentication** drop-down menu, ensure
         **NTLM** is selected.

      1. Select the **Enable Forms Based Authentication (FBA)** checkbox, in the
         **ASP.NET Membership provider name** box, type
         **FabrikamSqlMembershipProvider**, and in the **ASP.NET Role manager
         name** box, type **FabrikamSqlRoleProvider**.

      </div>

   1. In the **Application Pool**section:
      1. Click **Create a new application pool**.
      1. Under **Select a security account for this application pool**, click
         **Configurable**, and then select the service account specified in
         [Table 7](#Table_7_-_Web_applications) for the Fabrikam Extranet Web
         application.
   1. In the **Database Name and Authentication** section, in the **Database
      Name** box, type the corresponding value from Table 7.
   1. In the **Service Application Connections** section, ensure **default** is
      selected in the drop-down menu.
   1. Click **OK** to create the new Web application.

1. Wait for the Web application to be created and then click **OK**.

To create the initial site collections using SharePoint Central Administration:

1. On the Central Administration home page, in the **Application Management**
   section, click **Create site collections**.
1. On the **Create Site Collection** page, in the **Web Application** section,
   ensure the Fabrikam Extranet Web application is selected
   ([**http://extranet.fabrikam.com**](http://extranet.fabrikam.com)).
1. In the **Title and Description** section, type the title and description for
   the site collection using the corresponding values specified in
   [Table 8](#Table_8_-_Site_collections).
1. In the **Web Site Address** section, specify the path to use based on the
   value specified in Table 8.
1. In the **Template Selection** section, in the **Select a template** list,
   select the template specified in Table 8.
1. In the **Primary Site Collection Administrator** section, type the
   corresponding value specified in Table 8.
1. In the **Secondary Site Collection Administrator** section, type the username
   specified in Table 8.
1. In the **Quota Template** section, click the template in the **Select a quota
   template** list corresponding to the value specified in Table 8.
1. Click **OK**.
1. Wait for the site collection to be created and then click **OK**.
1. Create the remaining site collections specified in Table 8.

## Expand content database files {#Expand_content_database_files}

By default, SharePoint content database files are created with very small
initial sizes but with autogrowth enabled. Specifically, the data and log files
for content databases are created with initial sizes of 25 MB and 3 MB,
respectively. If these defaults were to be used on content databases expected to
grow significantly, performance would be severely impacted.

{{< div-block "note" >}}

> **Note**
>
> Skip this section for development environments configured with small VHD
> files.

{{< /div-block >}}

To increase the size of the database files:

1. Start SQL Server Management Studio and connect to the appropriate server.
1. In the **Object Explorer**, expand the **Databases** folder.
1. Right-click the **WSS_Content_FabrikamExtranet** database and then click
   **Properties**.
1. In the **Database Properties** dialog, in the **Select a page** area on the
   left, click **Files**.
1. Using the settings specified in
   [Table 9](#Table_9_-_Initial_data_and_log_file_sizes), type the new values
   for **Initial Size** and **Autogrowth**.
1. Click **OK**.

The following SQL statements can be used as an alternative to setting the sizes
through the Database Properties dialog:

```SQL
USE [master]
GO
ALTER DATABASE [WSS_Content_FabrikamExtranet]
  MODIFY FILE(
    NAME = N'WSS_Content_FabrikamExtranet'
    , SIZE = 10240000KB
    , FILEGROWTH = 512000KB)
GO
ALTER DATABASE [WSS_Content_FabrikamExtranet]
MODIFY FILE(
    NAME = N'WSS_Content_FabrikamExtranet_log'
    , SIZE = 409600KB
    , MAXSIZE = 4096000KB)
GO
```

## Configure object cache user accounts

The object cache in SharePoint Server 2010 performs queries using one of two
user accounts: the “Portal Super User” or the “Portal Super Reader.” These user
accounts must be properly configured to ensure the object cache works correctly.
The Portal Super User account must be an account that has Full Control access to
the Web application. The Portal Super Reader account must be an account that has
Full Read access to the Web application.

To configure object cache user accounts:

1. On the **Start** menu, click **All Programs**, click Microsoft SharePoint
   2010 Products, right-click **SharePoint 2010 Management Shell**, and then
   click **Run as administrator**. If prompted by User Account Control to allow
   the program to make changes to the computer, click **Yes**.
1. From the Windows PowerShell command prompt, change to the following
   directory:

   > **{build version or branch folder}\\[Source]\Deployment Files\Scripts**

1. Type the following command:

   ```PowerShell
   & '.\Configure Object Cache User Accounts.ps1'
   ```

1. Wait for the script to complete and verify no errors occurred during the
   process.
1. Type the following command to reset Internet Information Services (IIS):

   ```Batch
   iisreset
   ```

## Configure the People Picker to support searches across one-way trust

Use this procedure to enable selection of users and groups on the Fabrikam
Extranet from the internal domain (FABRIKAM).

{{< div-block "note" >}}

> **Note**
>
> A one-way trust relationship has been established from the extranet domain
> (EXTRANET) to the internal domain (FABRIKAM).

{{< /div-block >}}

To enable selection of people and groups from the internal Fabrikam domain:

1. Click **Start**, click **All Programs**, click **Accessories**, right-click
   **Command Prompt**, and then click **Run as administrator**.
1. Type the following command:

   ```Batch
   stsadm -o setapppassword -password {Key}
   ```

   {{< div-block "note" >}}

   > **Note**
   >
   > The key specified above can be any string. It is used to encrypt the
   > password specified in the following procedure when storing the credentials
   > in the database.
   >
   > This key is used to encrypt the password for the account used to access the
   > forest or domain. The encryption string must be the same for every server
   > in the farm.

   {{< /div-block >}}

1. Repeat the steps above on each Web server in the farm.
1. On one of the front-end Web servers, type the following command:

   ```Batch
   stsadm -o setproperty -pn peoplepicker-searchadforests -pv "domain:extranet.fabrikam.com,EXTRANET\svc-web-fabrikam,{password};domain:corp.fabrikam.com,FABRIKAM\svc-web-fabrikam,{password}" -url http://extranet.fabrikam.com
   ```

   {{< div-block "note" >}}

   > **Note**
   >
   > The usernames and passwords must not contain commas. After you run this
   > command, users can select users and groups from the listed forests and
   > domains from any front-end Web server in the farm.

   {{< /div-block >}}

## Configure SSL on the Internet zone

To add a public URL for HTTPS:

1. On the Central Administration home page, click **Application Management**.
1. On the **Application Management** page, in the **Web Applications** section,
   click **Configure alternate access mappings**.
1. On the **Alternate Access Mappings** page, click **Edit Public URLs**.
1. On the **Edit Public Zone URLs**page:
   1. In the **Alternate Access Mapping Collection** section, select the Web
      application specified in [Table 7](#Table_7_-_Web_applications).
   1. In the **Public URLs** section, copy the URL from the **Default** box to
      the **Internet** box, and change **http://** to **https://**.
   1. Click **Save**.

To add an HTTPS binding to the site in IIS:

1. Click **Start**, point to **Administrative Tools**, and then click **Internet
   Information Services (IIS) Manager**.
1. In Internet Information Services (IIS) Manager, click the plus sign (+) next
   to the server name that contains the Web application, and then click the plus
   sign next to **Sites** to view the Web applications that have been created.
1. Click the name of the Web application corresponding to the **Internet** zone
   (e.g. **SharePoint - extranet.fabrikam.com80**). In the **Actions** section,
   under the **Edit Site** heading, click **Bindings...**.
1. In the **Site Bindings** window, click **Add**.
1. In the **Add Site Binding**window:
   1. In the **Type:** dropdown, select **https**.
   1. In the **SSL Certificate:** dropdown, select the certificate corresponding
      to the site (e.g. extranet.fabrikam.com).
   1. Click **OK**.
   1. In the **Site Bindings** window, click **Close**.

## Enable anonymous access to the site

In addition to enabling anonymous access on the Web application, the root Web of
the site collection must also be configured to enable anonymous access. This can
be accomplished using a PowerShell script or through the corresponding
administration page on the site.

To enable anonymous access to the site using PowerShell:

1. On the **Start** menu, click **All Programs**, click Microsoft SharePoint
   2010 Products, right-click **SharePoint 2010 Management Shell**, and then
   click **Run as administrator**. If prompted by User Account Control to allow
   the program to make changes to the computer, click **Yes**.
1. From the Windows PowerShell command prompt, change to the following
   directory:

   > **{build version or branch folder}\\[Source]\Deployment Files\Scripts**

1. Type the following command:

   ```PowerShell
   & '.\Enable Anonymous Acess.ps1'
   ```

1. Wait for the script to complete and verify no errors occurred during the
   process.
1. Proceed to the next section
   ([Configure claims-based authentication](#Configure_claims-based_authentication)).

To enable anonymous access to the site using the site permissions page:

1. Browse to the home page of the site
   ([http://extranet.fabrikam.com](http://extranet.fabrikam.com)).
1. Click **Site Actions** and then click **Site Permissions**.
1. On the site permissions page, in the ribbon, click **Anonymous Access**.
1. In the **Anonymous Access** page, click **Entire Web site**, and then click
   **OK**.

## Configure claims-based authentication {#Configure_claims-based_authentication}

{{< div-block "note" >}}

> **Note**
>
> The Fabrikam Extranet solution:
>
> - Leverages the internal AD domain (FABRIKAM) for employees
> - Uses the extranet AD domain (EXTRANET) primarily for service accounts and
>   group policies for servers in the TEST and PROD farms
> - Stores membership and role information for customers, partners, suppliers,
>   and resellers in a SQL Server database (FabrikamDemo)

{{< /div-block >}}

In this section, claims-based authentication using a SQL Server database is
configured using the following high-level steps:

1. Create and configure the membership/role database
1. Modify the Web.config files for the following sites in order to support
   claims-based authentication:
   - SharePoint Central Administration v4
   - SecurityTokenServiceApplication
   - Fabrikam Extranet Web application
     ([http://extranet.fabrikam.com](http://extranet.fabrikam.com))
1. Create a user in the database using IIS Manager
1. Validate the configuration of the Web application

### Create and configure the membership/role database

In this step, the database for storing membership and role information for
external users (e.g. customers, partners, suppliers, and resellers) is created
and specific service accounts are added to the appropriate database roles.

To create the database used for storing membership and role information:

1. Click **Start**, point to **All Programs**, click **Accessories**, and
   right-click **Command Prompt**, and then click **Run as administrator**.
1. At the command prompt, type the following command:

   ```Batch
   cd %WinDir%\Microsoft.NET\Framework\v2.0.50727
   ```

1. Type the following command:

   ```Batch
   aspnet_regsql.exe
   ```

1. On the welcome page of the **ASP.NET SQL Server Setup Wizard**, click
   **Next**.
1. On the **Select a Setup Option** page, ensure the option to **Configure SQL
   Server for application services** is selected and then click **Next**.
1. On the **Select the Server and Database** page:

   1. In the **Server** box, type the name of the database server.
   1. Ensure the **Windows authentication** option is selected.
   1. In the **Database** dropdown list, type **FabrikamDemo**.
   1. Click **Next**.

1. On the **Confirm Your Settings** page, verify the settings, and then click
   **Next**.
1. Wait for the database to be created and then click **Finish**.

To add the service accounts to the membership/role database:

1. Start SQL Server Management Studio and connect to the appropriate server.
1. In **Object Explorer**, expand **Security**, and then expand **Logins**.
1. Right-click the login corresponding to the SharePoint farm service account
   (**EXTRANET\svc-sharepoint**) and then click **Properties**.
1. In the login properties dialog box:

   1. On the **User Mapping** page, in the **Users mapped to the login** list,
      click the checkbox for the membership/role database (**FabrikamDemo**),
      and then in the database role membership list, click the checkboxes for
      the following roles:

      <div class="overflow-auto">

      - **aspnet_Membership_BasicAccess**
      - **aspnet_Membership_ReportingAccess**
      - **aspnet_Roles_BasicAccess**
      - **aspnet_Roles_ReportingAccess**

      </div>

   1. Click **OK**.

1. Repeat the steps in this section to add the service account for the Fabrikam
   Extranet Web application (**EXTRANET\svc-web-fabrikam**) to the following
   roles:

   <div class="overflow-auto">

   - **aspnet_Membership_FullAccess**
   - **aspnet_Roles_BasicAccess**
   - **aspnet_Roles_ReportingAccess**

   </div>

{{< div-block "note important" >}}

> **Important**
>
> Database access must be granted to both the service account used for the Web
> application and the SharePoint farm account. If the SharePoint farm account
> does not have access to the database, the Security Token Service used for
> claims-based authentication will be unable to validate the credentials.

{{< /div-block >}}

{{< div-block "note" >}}

> **Note**
>
> The reason the database roles are different between the two service accounts
> is because the SharePoint farm account only needs permissions to validate
> credentials and determine role membership, whereas the Web application service
> account needs additional permissions in order to support other scenarios (e.g.
> "Change Password" and "Reset Password").

{{< /div-block >}}

### Add Web.config modifications for claims-based authentication

In order to complete the configuration of claims-based authentication, it is
necessary to modify the Web.config files for the following sites:

- SharePoint Central Administration v4
- Security Token Service
- Fabrikam Extranet
  ([http://extranet.fabrikam.com](http://extranet.fabrikam.com))

{{< div-block "note important" >}}

> **Important**
>
> These configuration changes must be completed on each SharePoint server in the
> farm.

{{< /div-block >}}

To configure the Central Administration Web.config file:

1. Click **Start**, point to **Administrative Tools**, and then click **Internet
   Information Services (IIS) Manager**.
1. In **Internet Information Services (IIS) Manager**, in the **Connections**
   pane, click the plus sign (+) next to the server name that contains the Web
   application, and then click the plus sign next to **Sites** to view the Web
   applications that have been created.
1. Right-click **SharePoint Central Administration v4**, and then click **Explore**. Windows Explorer opens, with the directories for the selected Web application listed.

   {{< div-block "note important" >}}

   > **Important**
   >
   > Before you make changes to the Web.config file, make a copy of it by using
   > a different name (for example, "Web - Copy.config"), so that if a mistake
   > is made in the file, you can delete it and use the original file.

   {{< /div-block >}}

1. Double-click the **Web.config** file to open the file.

   {{< div-block "note" >}}

   > **Note**
   >
   > If you see a dialog box that says that Windows cannot open the file, click
   > **Select the program from a list**, and then click **OK**. In the **Open
   > With** dialog box, click **Notepad**, and then click **OK**.

   {{< /div-block >}}

1. In the Web.config editor:

   1. After the end of the **/configuration/configSections** element (i.e. `</configSections>`), add the following elements:

      ```XML
      <connectionStrings>
        <add name="FabrikamDemo"
          connectionString="Server={databaseServer};Database=FabrikamDemo;Integrated Security=true" />
      </connectionStrings>
      ```

      {{< div-block-start "note important" >}}

      > **Important**
      >
      > Be sure to replace the **{databaseServer}** placeholder in the
      > connection string with the name of the database server.

      {{< div-block-end >}}

   1. Find the **/configuration/system.web/roleManager/providers** section and add the following elements:

      ```XML
      <add name="FabrikamSqlRoleProvider"
        type="System.Web.Security.SqlRoleProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a"
        applicationName="Fabrikam Demo Site"
        connectionStringName="FabrikamDemo" />
      ```

   1. Find the **/configuration/system.web/membership/providers** section and add the following elements:

      ```XML
      <add name="FabrikamSqlMembershipProvider"
        type="System.Web.Security.SqlMembershipProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a"
        applicationName="Fabrikam Demo Site"
        connectionStringName="FabrikamDemo"
        passwordFormat="Hashed" />
      ```

1. Save the changes to the Web.config file and close the editor.
1. Repeat the steps above on each Web server in the farm that hosts the Central
   Administration site.

To configure the Security Token Service Web.config file:

1. In **Internet Information Services (IIS) Manager**, in the **Connections** pane, expand the **SharePoint Web Services** site, right-click the **SecurityTokenServiceApplication** subsite, and then click **Explore**.

   {{< div-block "note important" >}}

   > **Important**
   >
   > Before you make changes to the Web.config file, make a copy of it by using
   > a different name (for example, "Web - Copy.config"), so that if a mistake
   > is made in the file, you can delete it and use the original file.

   {{< /div-block >}}

1. Double-click the **Web.config** file to open the file.
1. In the Web.config editor, add the following elements to the `<configuration>` root element:

   ```XML
   <connectionStrings>
     <add
     name="FabrikamDemo"
     connectionString="Server={databaseServer};Database=FabrikamDemo;Integrated Security=true" />
   </connectionStrings>

   <system.web>
     <membership>
       <providers>
         <add
         name="FabrikamSqlMembershipProvider"
         type="System.Web.Security.SqlMembershipProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a"
         applicationName="Fabrikam Demo Site"
         connectionStringName="FabrikamDemo"
         passwordFormat="Hashed" />
       </providers>
     </membership>
     <roleManager>
       <providers>
         <add
         name="FabrikamSqlRoleProvider"
         type="System.Web.Security.SqlRoleProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a"
         applicationName="Fabrikam Demo Site"
         connectionStringName="FabrikamDemo" />
       </providers>
     </roleManager>
   </system.web>
   ```

   {{< div-block "note important" >}}

   > **Important**
   >
   > Be sure to replace the **{databaseServer}** placeholder in the connection
   > string with the name of the database server.

   {{< /div-block >}}

1. Save the changes to the Web.config file and close the editor.
1. Repeat the steps above on each Web server in the farm.

To configure the Web.config file for the Fabrikam Extranet Web application:

1. In **Internet Information Services (IIS) Manager**, in the **Connections** pane, right-click the Web application (e.g. **SharePoint - extranet.fabrikam.com80**), and then click **Explore**.

   {{< div-block "note important" >}}

   > **Important**
   >
   > Before you make changes to the Web.config file, make a copy of it by using
   > a different name (for example, "Web - Copy.config"), so that if a mistake
   > is made in the file, you can delete it and use the original file.

   {{< /div-block >}}

1. Double-click the **Web.config** file to open the file.
1. In the Web.config editor:

   1. After the end of the **/configuration/configSections** element (i.e. `</configSections>`), add the following elements:

      ```XML
      <connectionStrings>
        <add name="FabrikamDemo"
          connectionString="Server={databaseServer};Database=FabrikamDemo;Integrated Security=true" />
      </connectionStrings>
      ```

      {{< div-block-start "note important" >}}

      > **Important**
      >
      > Be sure to replace the **{databaseServer}** placeholder in the
      > connection string with the name of the database server.

      {{< div-block-end >}}

   1. Find the **/configuration/system.web/roleManager/providers** section and add the following elements:

      ```XML
      <add name="FabrikamSqlRoleProvider"
        type="System.Web.Security.SqlRoleProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a"
        applicationName="Fabrikam Demo Site"
        connectionStringName="FabrikamDemo" />
      ```

      {{< div-block-start "note warning" >}}

      > **Warning**
      >
      > Do not overwrite any existing entries in this Web.config file.

      {{< div-block-end >}}

   1. Find the **/configuration/system.web/membership/providers** section and add the following elements:

      ```XML
      <add name="FabrikamSqlMembershipProvider"
        type="System.Web.Security.SqlMembershipProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a"
        applicationName="Fabrikam Demo Site"
        connectionStringName="FabrikamDemo"
        enablePasswordReset="true"
        enablePasswordRetrieval="false"
        passwordFormat="Hashed"
        requiresQuestionAndAnswer="true"
        requiresUniqueEmail="true" />
      ```

1. Save the changes to the Web.config file and close the editor.
1. Repeat the steps above on each Web server in the farm.

### Create a user in the database using IIS Manager

To create a user for the Fabrikam Extranet:

1. In **Internet Information Services (IIS) Manager**, click the Fabrikam
   Extranet Web application (e.g. **SharePoint -- extranet.fabrikam.com80**) and
   then double-click **.NET Users**.
1. When prompted with an error stating the feature cannot be used because the
   default provider is not a trusted provider, click **OK**.
1. In the **Actions** pane, click **Set Default Provider...**
1. In the **Edit .NET Users Settings** dialog box, note that the default
   provider configured in SharePoint Server 2010 is "i". In the **Default
   Provider** list, click **FabrikamSqlMembershipProvider**, and then click
   **OK**.
1. In the **Actions** pane, click **Add...**
1. When prompted with an error stating the default .NET Roles provider does not
   exist, click **OK**.
1. In the **Add .NET User**dialog:
   1. On the **.NET User Account Details** page, type the appropriate values in
      the **User Name**, **E-mail**, **Password**, **Confirm Password**,
      **Question**, and **Answer** boxes, and then click **Next**.
   1. On the **.NET User Roles** page, click **Finish**.
1. In the **Actions** pane, click **Set Default Provider...**
1. In the **Edit .NET Users Settings** dialog box, in the **Default Provider**
   list, click **i**, and then click **OK**.

### Validate claims authentication configuration

At this point in the installation, claims authentication for the Fabrikam
Extranet Web application has been configured and a new site collection has been
created based on the Publishing Portal template.

The site should resemble the following:

{{< anchor name="Figure_3:_Default_home_page_for_“Publishing_Portal”_site_in_SharePoint_Server_2010" >}}

{{< figure
  src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/SharePoint-Server-2010-Publishing-Portal-600x374.png"
  alt="Default home page for “Publishing Portal” site in SharePoint Server 2010"
  class="screenshot" height="374" width="600"
  caption="Figure 3: Default home page for “Publishing Portal” site in SharePoint Server 2010" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/SharePoint-Server-2010-Publishing-Portal-1006x627.png)

Although the site does not yet have the Fabrikam branding or any custom code
deployed, it is still possible to validate the basic functionality of the site
(most notably that claims authentication is working as expected).

The steps in this section validate the Web application works as expected when
using both Forms-Based Authentication and Windows authentication.

To login to the website using forms authentication:

1. Browse to the home page page the Fabrikam Extranet website
   ([http://extranet.fabrikam.com](http://extranet.fabrikam.com)) and click
   **Sign In**.
1. On the **Sign In**page:
   1. In the dropdown list, click **Forms Authentication**.
   1. When prompted to enter the **User name** and **Password**, type the
      credentials specified in the previous step and then click **Sign In**.
1. Verify the home page is displayed and the **Sign In** link has been replaced
   with the "Welcome" menu.

To login to the website using Windows authentication:

1. Add the Fabrikam Extranet website to the **Local intranet** zone (in order to seamlessly authenticate with the current domain credentials).

   {{< div-block-start "note" >}}

   > **Note**
   >
   > This is discussed in more detail in the following blog post:
   >
   > {{< reference title="Be \"In the Zone\" to Avoid Entering Credentials" linkHref="/blog/jjameson/2007/03/22/be-in-the-zone-to-avoid-entering-credentials" linkText="https://www.technologytoolbox.com/blog/jjameson/2007/03/22/be-in-the-zone-to-avoid-entering-credentials" >}}

   {{< div-block-end >}}

1. Browse to the home page page the Fabrikam Extranet website
   ([http://extranet.fabrikam.com](http://extranet.fabrikam.com)) and click
   **Sign In**.
1. On the **Sign In** page, in the dropdown list, click **Windows
   Authentication**.
1. Verify the home page is displayed and the **Sign In** link has been replaced
   with the "Welcome" menu.

## Enable disk-based caching for the Web application

By default, the disk-based BLOB cache is off, but it is strongly recommended to
enable disk-based caching in order to vastly reduce the number of database
roundtrips for each and every page request on your site.

This is discussed in more detail in the following blog post:

{{< reference title="Always Enable Disk-Based Caching in SharePoint Server 2010" linkHref="/blog/jjameson/2010/11/16/always-enable-disk-based-caching-in-sharepoint-server-2010" linkText="https://www.technologytoolbox.com/blog/jjameson/2010/11/16/always-enable-disk-based-caching-in-sharepoint-server-2010" >}}

Use the following procedure to configure disk-based cache settings for a Web
application.

{{< div-block "note important" >}}

> **Important**
>
> The following steps must be performed on each front-end Web server in the farm
> and for each Web.config file used by the Web application (i.e. the default
> Web.config as well as the Web.config for each Web application configured as an
> Alternate Access Mapping for the **Default** zone).

{{< /div-block >}}

{{< div-block "note" >}}

> **Note**
>
> Disk-based caching does not need to be enabled in development environments.
> However it should always be enabled in both TEST and PROD.

{{< /div-block >}}

To configure BLOB cache settings:

1. Click **Start**, point to **Administrative Tools**, and then click **Internet
   Information Services (IIS) Manager**.
1. In **Internet Information Services (IIS) Manager**, in the **Connections**
   pane, click the plus sign (+) next to the server name that contains the Web
   application, and then click the plus sign next to **Sites** to view the Web
   applications that have been created.
1. Right-click the name of the Web application to configure the disk-based cache for, and then click **Explore**. Windows Explorer opens, with the directories for the selected Web application listed.

   {{< div-block "note important" >}}

   > **Important**
   >
   > Before you make changes to the Web.config file, make a copy of it by using
   > a different name (for example, “Web - Copy.config”), so that if a mistake
   > is made in the file, you can delete it and use the original file.

   {{< /div-block >}}

1. Double-click the **Web.config** file to open the file.

   {{< div-block "note" >}}

   > **Note**
   >
   > If you see a dialog box that says that Windows cannot open the file, click
   > **Select the program from a list**, and then click **OK**. In the **Open
   > With** dialog box, click **Notepad**, and then click **OK**.

   {{< /div-block >}}

1. If the Windows dialog box appears, select **Select a program from a list of
   installed programs**, and then click **OK**. In the **Open With** dialog box,
   click **Notepad**, and then click **OK**.
1. In the Web.config editor, find the following line:

   ```XML
   <BlobCache location="C:\BlobCache\14" path="...(gif|jpg|jpeg|...)$" maxSize="10"
       enabled="false" />
   ```

1. In this line, change the **location** attribute to specify a directory that has enough space to accommodate the cache size.

   {{< div-block "note" >}}

   > **Note**
   >
   > It is strongly recommended to specify a directory that is not on the same
   > drive as where either the server operating system swap files or server log
   > files are stored.

   {{< /div-block >}}

1. To add or remove file types from the list of file types to be cached, for the
   **path** attribute, modify the regular expression to include or remove the
   appropriate file extension. If you add file extensions, make sure to separate
   each file type with a pipe (|), as shown above.
1. To change the size of the cache, type a new number for **maxSize**. The size is expressed in gigabytes (GB), and 10 GB is the default.

   {{< div-block "note important" >}}

   > **Important**
   >
   > It is recommended that you not set the cache size smaller than 10 GB. When
   > you set the cache size, make sure to specify a number large enough to
   > provide a buffer at least 20 percent bigger than the estimated size of the
   > content that will be stored in the cache.

   {{< /div-block >}}

1. To enable the BLOB cache, change the **enabled** attribute to **true**.
1. Save the file, and then close it.

## Configure SharePoint groups

To add members to SharePoint groups:

1. Browse to the home page of the site
   ([http://extranet.fabrikam.com](http://extranet.fabrikam.com)).
1. Click the **Sign In** link and sign in using **Windows Authentication**.
1. On the home page of the site, click **Site Actions** and then click **Site
   Permissions**.
1. On the site permissions page, click the SharePoint group specified in
   [Table 10](#Table_10_-_SharePoint_groups_and_permissions_used_for_entitlement).
1. On the **People and Groups - {Group Name}** page, in the toolbar, click
   **New**, and then click **Add Users**.
1. In the **Grant Permissions**dialog window:
   1. In the **Select Users** section, type the group member(s) specified in
      Table 10. You can also click the applicable icon to check a name or browse
      for users and groups.
   1. Click **OK**.
1. Repeat the previous steps for the remaining SharePoint groups specified in
   Table 10.

# Configure service applications

## Configure the State Service

The State Service is required in order to use the out-of-the-box workflows in
SharePoint Server 2010 (e.g. Approval - SharePoint 2010) or any other features
that leverage InfoPath Forms Services.

When using the Farm Configuration Wizard to configure the State Service, the
resulting database is named StateService\_{GUID}. In order to avoid lengthy
database names containing GUIDs, the State Service is configured using
PowerShell.

To configure the State Service:

1. On the **Start** menu, click **All Programs**, click Microsoft SharePoint
   2010 Products, right-click **SharePoint 2010 Management Shell**, and then
   click **Run as administrator**. If prompted by User Account Control to allow
   the program to make changes to the computer, click **Yes**.
1. From the Windows PowerShell command prompt, change to the following
   directory:

   > **{build version or branch folder}\\[Source]\Deployment Files\Scripts**

1. Type the following command:

   ```PowerShell
   & '.\Configure State Service.ps1'
   ```

1. Wait for the script to complete and verify no errors occurred during the
   process.

## Create and configure the Search Service Application

When using the Farm Configuration Wizard to create and configure the Search
Service Application, the resulting databases are named
Search_Service_Application_DB\_{GUID},
Search_Service_Application_CrawlDB\_{GUID}, and
Search_Service_Application_PropertyStoreDB\_{GUID}.

In order to avoid the lengthy database names containing GUIDs, the Search
Service Application is created and configured using PowerShell.

To create and configure the Search Service Application:

1. On the **Start** menu, click **All Programs**, click **Microsoft SharePoint
   2010 Products**, right-click **SharePoint 2010 Management Shell**, and then
   click **Run as administrator**. If prompted by User Account Control to allow
   the program to make changes to the computer, click **Yes**.
1. From the Windows PowerShell command prompt, change to the following
   directory:

   > **{build version or branch folder}\\[Source]\Deployment Files\Scripts**

1. Type the following command:

   ```PowerShell
   & '.\Configure SharePoint Search.ps1'
   ```

1. When prompted for default content access account credentials, verify the user
   name (EXTRANET\svc-index), type the password, and then click **OK**.
1. If prompted for the service application credentials, verify the user name
   (EXTRANET\svc-spserviceapp), type the password, and then click **OK**.
1. Wait for the script to complete and verify no errors occurred during the process.

   {{< div-block "note" >}}

   > **Note**
   >
   > It may take several minutes to create and configure the Search Service
   > Application.

   {{< /div-block >}}

## Configure the search crawl schedules

Use the following procedure to configure the crawl schedules for the default
content source.

To configure the crawl schedules:

1. On the Central Administration home page, in the **Application Management**
   section, click **Manage service applications**.
1. On the **Service Applications** tab, click **Search Service Application**
   (where the **Type** column is **Search Service Application**).
1. On the **Search Administration** page, in the **Crawling** section, click
   **Content Sources**.
1. On the **Manage Content Sources** page, click **Local SharePoint sites** to
   edit the content source.
1. On the **Edit Content Source** page, in the **Crawl Schedules** section,
   under **Full Crawl**, click **Create schedule**.
1. On the **Manage Schedules** page, configure the type of schedule and the
   schedule settings specified in [Table 11](#Table_11_-_Search_schedules), and
   then click **OK**.
1. On the **Edit Content Source** page, in the **Crawl Schedules** section,
   under **Incremental Crawl**, click **Create schedule**.
1. On the **Manage Schedules** page, configure the type of schedule and the
   schedule settings specified in Table 11, and then click **OK**.
1. On the **Edit Content Source** page, in the **Start Full Crawl** section,
   click the **Start full crawl of this content source** checkbox.
1. Click **OK**.

# Install and configure Office Web Apps

## Install Office Web Apps

In this section, the Office Web Apps are installed from the installation source.

{{< div-block "note important" >}}

> **Important**
>
> This task must be completed on each SharePoint server in the farm.

{{< /div-block >}}

To install Office Web Apps:

1. From the Office Web Apps installation location, double-click the appropriate
   executable file.
1. On the **Enter your Product Key** page, enter the corresponding product key,
   and then click **Continue**.
1. Review the licensing agreement. If you accept the terms and conditions,
   select **I accept the terms of this agreement**, and then click **Continue**.
1. On the **Choose a file location** tab, change the path for the search index
   files as specified in [Table 3](#Table_3_-_Installation_file_locations), and
   then click **Install Now**.
1. Wait for the installation to finish.
1. On the **Run Configuration Wizard** page, clear the **Run the SharePoint Products and Technologies Configuration Wizard now** checkbox, and then click **Close**.

   {{< div-block "note" >}}

   > **Note**
   >
   > For consistency of approach, it is recommended that you do not run the
   > configuration wizard until Office Web Apps has been installed on all
   > application and front-end Web servers in the farm.

   {{< /div-block >}}

1. Repeat the steps above on each SharePoint server in the farm.

## Run PSConfig to register Office Web Apps services

In this task the Office Web Apps services are registered on the SharePoint
server.

{{< div-block "note important" >}}

> **Important**
>
> This task must be completed on each SharePoint server in the farm.

{{< /div-block >}}

To run PSConfig to register the services:

1. Click **Start**, point to **All Programs**, click **Microsoft SharePoint 2010
   Products**, and then click **SharePoint 2010 Products Configuration Wizard**.
   If prompted by User Account Control to allow the program to make changes to
   the computer, click **Yes**.
1. On the **Welcome to SharePoint Products** page, click **Next**.
1. In the dialog box that notifies you that some services may have to be started
   or reset during configuration, click **Yes**.
1. On the **Completing the SharePoint Products Configuration Wizard** page,
   click **Next**.
1. Wait for the configuration to complete.
1. On the **Configuration Successful** page, click **Finish**.
1. Repeat the steps above on each SharePoint server in the farm.

## Start the Office Web Apps service instances and create service applications

A service instance provides the physical location for a service application. The
service instances must be started for each server that will run the Office Web
Apps service applications.

To start the service instances by using Central Administration:

1. On the Central Administration home page, under **System Settings**, click
   **Manage services on server**.
1. On the **Services on server** page, in the **Server** control, select a
   server, and then start **Excel Calculation Services**, **PowerPoint
   Service**, and **Word Viewing Service**. Repeat this step for each SharePoint
   server in the farm.

To create the service applications and proxies:

1. On the Central Administration home page, click **Configuration Wizards**.
1. On the **Configuration Wizards** page, click **Launch the Farm Configuration
   Wizard**.
1. On the **Configure your SharePoint Farm** welcome page, click **Start the
   Wizard**.
1. On the **Configure your SharePoint Farm**page:
   1. In the **Service Account** section, click **Use existing managed
      account**, and then select **EXTRANET\svc-spserviceapp** in the drop-down
      list.
   1. In the **Services** section, clear all enabled checkboxes _except_ for
      **Excel Services Application**, **PowerPoint Service Application**, and
      **Word Viewing Service**.
   1. Click **Next**.
1. When prompted to create a new top-level Web site, click **Skip**.
1. On the **Configure your SharePoint Farm** completion page, click **Finish**.

## Configure Excel Services Application trusted location

When the Excel Services Application is created, a default trusted location is
automatically configured (**http://**) for all content on the SharePoint farm.
This default trusted location enables any file to be loaded from the SharePoint
farm or stand-alone deployment on Excel Services. However, this default trusted
location does not support HTTPS (https://) and therefore results in the
following error when attempting to access an Excel workbook on a secured
connection:

{{< div-block "errorMessage" >}}

> This workbook cannot be opened because it is not stored in an Excel Services
> Application trusted location.

{{< /div-block >}}

Use the following procedure to change the default trusted location to support
HTTPS.

{{< div-block "note important" >}}

> **Important**
>
> Skip this section for environments that are not configured with SSL
> certificates (e.g. development environments).

{{< /div-block >}}

To configure the Excel Services Application trusted location for HTTPS instead
of HTTP:

1. On the Central Administration home page, in the **Application Management**
   section, click **Manage service applications**.
1. On the **Service Applications** tab, click **Excel Services Application**
   (where the **Type** column is **Excel Services Application Web Service
   Application**).
1. On the **Manage Excel Services Application** page, click **Trusted File
   Locations**.
1. On the **Excel Services Application Trusted File Locations** page, click the
   default trusted file location (**http://**) to edit the corresponding
   settings.
1. On the **Excel Services Application Edit Trusted File Location** page, in the **Location** section, change the **Address** from **http://** to **https://** and then click **OK**.

   {{< div-block "note" >}}

   > **Note**
   >
   > Since users are automatically redirected from http:// to https:// during
   > sign in, it is not expected that Excel Services will be used over HTTP
   > (only HTTPS). If it is necessary to support both HTTP and HTTPS, then a
   > separate trusted file location will need to be configured.

   {{< /div-block >}}

## Configure the Office Web Apps cache

By default, when you install Office Web Apps, the cache available to render
documents is 100 GB and the cache expiration period is 30 days. Also note that
cached content for Office Web Apps is stored in a SharePoint content database.

To configure the Office Web Apps cache and create a separate content database
for caching:

1. On the **Start** menu, click **All Programs**, click Microsoft SharePoint 2010 Products, right-click **SharePoint 2010 Management Shell**, and then click **Run as administrator**. If prompted by User Account Control to allow the program to make changes to the computer, click **Yes**.

   {{< div-block "note important" >}}

   > **Important**
   >
   > You must start a new instance of the SharePoint 2010 Management Shell after
   > installing Office Web Apps in order for the new PowerShell cmdlets to be
   > recognized (e.g. **Set-SPOfficeWebAppsCache**).

   {{< /div-block >}}

1. From the Windows PowerShell command prompt, change to the following
   directory:

   > **{build version or branch folder}\\[Source]\Deployment Files\Scripts**

1. Type the following command:

   ```PowerShell
   & '.\Configure Office Web Apps Cache.ps1'
   ```

1. Wait for the script to complete and verify no errors occurred during the process.

   {{< div-block "note" >}}

   > **Note**
   >
   > The cache site collection is created via a SharePoint timer job. Consequently you may encounter the following error (depending on how quickly the installation steps are performed):
   >
   > {{< div-block-start "errorMessage" >}}
   >
   > > Set-SPOfficeWebAppsCache : Specified web application doesn't exist or
   > > doesn't have a cache site collection.
   >
   > {{< div-block-end >}}
   > If this error occurs, wait a few minutes and then run the script again.

   {{< /div-block >}}

1. Type the following command to reset Internet Information Services (IIS):

   ```Batch
   iisreset
   ```

To increase the size of the database files for the Office Web Apps cache:

1. Start SQL Server Management Studio and connect to the appropriate server.
1. In the **Object Explorer**, expand the **Databases** folder.
1. Right-click the **OfficeWebAppsCache** database and then click
   **Properties**.
1. In the **Database Properties** dialog, in the **Select a page** area on the
   left, click **Files**.
1. Using the settings specified in
   [Table 9](#Table_9_-_Initial_data_and_log_file_sizes), specify the new values
   for **Initial Size** and **Autogrowth**.
1. Click **OK**.

The following SQL statements can be used as an alternative to setting the sizes
through the Database Properties dialog:

```SQL
USE [master]
GO
ALTER DATABASE [OfficeWebAppsCache]
  MODIFY FILE(
    NAME = N'OfficeWebAppsCache'
    , SIZE = 10240000KB
    , FILEGROWTH = 512000KB)
GO
ALTER DATABASE [OfficeWebAppsCache]
  MODIFY FILE(
    NAME = N'OfficeWebAppsCache_log'
    , SIZE = 409600KB
    , MAXSIZE = 4096000KB)
GO
```

## Grant access to the Web application content database for Office Web Apps

Since the service account used to run the Office Web Apps service applications
is different from the account used to run the application pool for the Web
application, it is necessary to explicitly grant access to the content database.

To grant the Office Web Apps service account access to the content database:

1. On the **Start** menu, click **All Programs**, click **Microsoft SharePoint
   2010 Products**, right-click **SharePoint 2010 Management Shell**, and then
   click **Run as administrator**. If prompted by User Account Control to allow
   the program to make changes to the computer, click **Yes**.
1. From the Windows PowerShell command prompt, type the following commands:

   ```PowerShell
   $webApp = Get-SPWebApplication -Identity "http://extranet.fabrikam.com"

   $webApp.GrantAccessToProcessIdentity("EXTRANET\svc-spserviceapp")
   ```

# Deploy the Fabrikam Extranet solution and create partner sites

## Configure logging

The Fabrikam Extranet needs to be configured to log errors, warnings, and
informational messages to the Event Log. The events are logged to the
**Application** event log with a **Source** value of **Fabrikam Demo Site**.
Since the service account used for the Web application does not have permission
to create custom a event source, the **Fabrikam Demo Site** event source must be
created by an administrator.

{{< div-block "note important" >}}

> **Important**
>
> This task must be completed on each SharePoint server in the farm.

{{< /div-block >}}

To add the new event source:

1. On the **Start** menu, click **All Programs**, click **Microsoft SharePoint
   2010 Products**, right-click **SharePoint 2010 Management Shell**, and then
   click **Run as administrator**. If prompted by User Account Control to allow
   the program to make changes to the computer, click **Yes**.
1. From the Windows PowerShell command prompt, change to the following
   directory:

   > **{build version or branch folder}\\[Source]\Deployment Files\Scripts**

1. Type the following command:

   ```PowerShell
   & '.\Add Event Log Sources.ps1'
   ```

1. Wait for the script to complete and verify no errors occurred during the
   process.
1. Repeat the steps above on each SharePoint server in the farm.

## Install Fabrikam SharePoint solutions and activate the features

To install and activate the features:

1. On the **Start** menu, click **All Programs**, click **Microsoft SharePoint
   2010 Products**, right-click **SharePoint 2010 Management Shell**, and then
   click **Run as administrator**. If prompted by User Account Control to allow
   the program to make changes to the computer, click **Yes**.
1. From the Windows PowerShell command prompt, change to the following
   directory:

   > **{build version or branch folder}\\[Source]\Deployment Files\Scripts**

1. Type the following command:

   ```PowerShell
   & '.\Add Solutions.ps1'
   ```

1. Wait for the solutions to be added and then type the following command:

   ```PowerShell
   & '.\Deploy Solutions.ps1'
   ```

1. Wait for the solutions to be deployed and then type the following command:

   ```PowerShell
   & '.\Activate Features.ps1'
   ```

1. Wait for the feature activations to complete, and then minimize or close the
   PowerShell command prompt.

At this point, the home page of the Fabrikam Extranet site should resemble the
screenshot shown in [Figure 4](#Figure_4:_Fabrikam_Extranet_home_page) (but
without the **Sample Style Guide** and **Reusable Content Sample** links).

{{< anchor name="Figure_4:_Fabrikam_Extranet_home_page" >}}

{{< figure src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/Fabrikam-Extranet-home-page-600x277.png" alt="Fabrikam Extranet home page" class="screenshot" height="277" width="600" caption="Figure 4: Fabrikam Extranet home page" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/Fabrikam-Extranet-home-page-1010x467.png)

## Configure sample content (optional)

The **Sample Style Guide** and **Reusable Content Sample** links shown in
[Figure 4](#Figure_4:_Fabrikam_Extranet_home_page) are created by the "Test
Console" utility in the Fabrikam solution.

To create the sample content:

1. From a command prompt, change to the directory containing the "Test Console" utility.

   {{< div-block "note" >}}

   > **Note**
   >
   > If deploying a build copied from the release server (i.e. DAZZLER), use the
   > following path:
   >
   > > **{build version}\\{Debug|Release}**
   >
   > If deploying a build to a local development environment from a TFS
   > workspace, use the following path:
   >
   > > **{branch folder}\Source\Tools\TestConsole\bin\\{Debug|Release}**

   {{< /div-block >}}

1. Type the following command:

   ```Batch
   Fabrikam.Demo.Tools.TestConsole.exe
   ```

1. Wait for the program to complete and verify no errors occurred during the
   process.

## Create and configure a partner site

### Create site collection for a Fabrikam partner

A site collection for a Fabrikam partner can either be created using a
PowerShell script or through Central Administration.

To create a site collection for a Fabrikam partner using the PowerShell script:

1. On the **Start** menu, click **All Programs**, click **Microsoft SharePoint
   2010 Products**, right-click **SharePoint 2010 Management Shell**, and then
   click **Run as administrator**. If prompted by User Account Control to allow
   the program to make changes to the computer, click **Yes**.
1. From the Windows PowerShell command prompt, change to the following
   directory:

   > **{build version or branch folder}\\[Source]\Deployment Files\Scripts**

1. Run the **Create Partner Site Collection.ps1** script and provide the name of the partner, for example:

   ```PowerShell
   & '.\Create Partner Site Collection.ps1' "Contoso Shipping"
   ```

1. Wait for the script to complete and verify no errors occurred during the
   process.
1. Proceed to the next section
   ([Apply the “Fabrikam Partner Site” template to the top-level site](#Apply_custom_site_template)).

To create a site collection for a Fabrikam partner using Central Administration:

<div class="overflow-auto">

1. On the Central Administration home page, in the **Application Management**
   section, click **Create site collections**.
1. On the **Create Site Collection** page, in the **Web Application** section,
   ensure the Fabrikam Extranet application is selected
   ([http://extranet.fabrikam.com](http://extranet.fabrikam.com)).
1. In the **Title and Description** section, in the **Title** box, type the name
   of the partner (e.g. “Contoso Shipping”).
1. In the **Web Site Address** section, specify the path to use based on the
   name of the partner (it is recommended to replace spaces in the partner name
   with dashes).
1. In the **Template Selection** section, in the **Select a template** list,
   click the **Custom** tab and then click **&lt; Select template later...
   &gt;**.
1. In the **Primary Site Collection Administrator** section, type the name of
   the user who should have administrator permissions on the site collection.
1. In the **Secondary Site Collection Administrator** section, type the name of
   another user who should have administrator permissions on the site
   collection.
1. Click **OK**.
1. Wait for the site collection to be created and then click **OK**.
1. Browse to the new site collection (e.g.
   [http://extranet.fabrikam.com/sites/Contoso-Shipping](http://extranet.fabrikam.com/sites/Contoso-Shipping)).
   Since no template has been applied to the top-level site, the **Template
   Selection** page is displayed.
1. On the **Template Selection** page, click **Site Actions**, and then click
   **Site Settings**.
1. On the **Site Settings** page, in the **Site Collection Administration**
   section, click **Site collection features**.
1. On the site collection **Features**page, activate the following features:
   1. **Fabrikam Demo - Publishing Layouts**
   1. **Fabrikam Demo - Web Templates**

</div>

### Apply the “Fabrikam Partner Site” template to the top-level site {#Apply_custom_site_template}

To apply the custom site template to the top-level site in the site collection:

<div class="overflow-auto">

1. Browse to the new site collection (e.g.
   [http://extranet.fabrikam.com/sites/Contoso-Shipping](http://extranet.fabrikam.com/sites/Contoso-Shipping)).
   Since no template has been applied to the top-level site, the **Template
   Selection** page is displayed.
1. On the **Template Selection**page:
   1. In the **Template Selection** section, click the **Fabrikam** tab, and
      then click the **Fabrikam Partner Site** template.
   1. Click **OK**.
1. On the **Set Up Groups for this Site** page, click **OK** to create the
   default groups.

</div>

### Update the partner site home page

Edit the site home page to:

- Add the partner logo and change the default “welcome” text
- Add the Web Part to show items in the **Shared Documents** library
- Remove or replace the page image
- Remove the **Getting Started** text

## Create and configure a team collaboration site

### Create the team collaboration site

To create a team collaboration site for a partner site:

<div class="overflow-auto">

1. Browse to the partner site collection (e.g.
   [http://extranet.fabrikam.com/sites/Contoso-Shipping](http://extranet.fabrikam.com/sites/Contoso-Shipping)).
1. On the home page of the partner site, click **Site Actions**, and then click
   **New Site**.
1. In the dialog window for creating a new site:
   1. In the **Filter By** section, click **Fabrikam**, and then click
      **Fabrikam Team Site**.
   1. Type the title and URL name for the new site (e.g. Finance), and then
      click **More Options**.
   1. In the **Permissions** section, click **Use unique permissions**.
   1. In the **Navigation Inheritance** section, click **Yes** to use the top
      link bar from the parent site.
   1. Click **Create**.
   1. On the **Set Up Groups for this Site** page, in the **Visitors to this
      Site** section, click **Create a new group** and then click **OK**.

</div>

### Update the team site home page

Edit the site home page to:

- Add the partner logo and change the default “welcome” text
- Add the Web Part to show items in the **Shared Documents** library
- Remove or replace the page image
- Remove the **Getting Started** text

{{< div-block-start "appendix no-numbered-headings" >}}

# Appendix A - Planning worksheets {#Appendix-A-Planning-worksheets}

There are a number of configuration parameters that need to be determined before
installing the Fabrikam Extranet solution. The various parameters and
configuration settings are listed in the following tables.

<a name="Table_2_-_Service_accounts"></a>

<div class="d-sm-none">
  <a href='{{< relref "resources/table-2-popout" >}}' target="_blank">Table 2 - Service accounts</a>
  {{< svg-icon "arrow-up-right-square" >}}
  <p>(Insufficient width to show table content here.)</p>
</div>
<div class="d-none d-sm-block">
   {{< include-html "resources/table-2.html" >}}
</div>

<a name="Table_3_-_Installation_file_locations"></a>
{{< include-html "resources/table-3.html" >}}

<a name="Table_4_-_Diagnostic_logging"></a>

<div class="d-sm-none">
  <a href='{{< relref "resources/table-4-popout" >}}' target="_blank">Table 4 - Diagnostic logging</a>
  {{< svg-icon "arrow-up-right-square" >}}
  <p>(Insufficient width to show table content here.)</p>
</div>
<div class="d-none d-sm-block">
   {{< include-html "resources/table-4.html" >}}
</div>

<a name="Table_5_-_Web_analytics_and_health_data_collection"></a>
{{< include-html "resources/table-5.html" >}}

<a name="Table_6_-_Outgoing_e-mail_settings"></a>

<div class="d-md-none">
  <a href='{{< relref "resources/table-6-popout" >}}' target="_blank">Table 6 - Outgoing e-mail settings</a>
  {{< svg-icon "arrow-up-right-square" >}}
  <p>(Insufficient width to show table content here.)</p>
</div>
<div class="d-none d-md-block">
   {{< include-html "resources/table-6.html" >}}
</div>

<a name="Table_7_-_Web_applications"></a>

<div class="d-lg-none">
  <a href='{{< relref "resources/table-7-popout" >}}' target="_blank">Table 7 - Web applications</a>
  {{< svg-icon "arrow-up-right-square" >}}
  <p>(Insufficient width to show table content here.)</p>
</div>
<div class="d-none d-lg-block">
   {{< include-html "resources/table-7.html" >}}
</div>

<a name="Table_8_-_Site_collections"></a>

<div class="d-lg-none">
  <a href='{{< relref "resources/table-8-popout" >}}' target="_blank">Table 8 - Site collections</a>
  {{< svg-icon "arrow-up-right-square" >}}
  <p>(Insufficient width to show table content here.)</p>
</div>
<div class="d-none d-lg-block">
   {{< include-html "resources/table-8.html" >}}
</div>

<a name="Table_9_-_Initial_data_and_log_file_sizes"></a>

<div class="d-lg-none">
  <a href='{{< relref "resources/table-9-popout" >}}' target="_blank">Table 9 - Initial data and log file sizes</a>
  {{< svg-icon "arrow-up-right-square" >}}
  <p>(Insufficient width to show table content here.)</p>
</div>
<div class="d-none d-lg-block">
   {{< include-html "resources/table-9.html" >}}
</div>

<a name="Table_10_-_SharePoint_groups_and_permissions_used_for_entitlement"></a>

<div class="d-sm-none">
  <a href='{{< relref "resources/table-10-popout" >}}' target="_blank">Table 10 - SharePoint groups and permissions used for entitlement</a>
  {{< svg-icon "arrow-up-right-square" >}}
  <p>(Insufficient width to show table content here.)</p>
</div>
<div class="d-none d-sm-block">
   {{< include-html "resources/table-10.html" >}}
</div>

<a name="Table_11_-_Search_schedules"></a>
{{< include-html "resources/table-11.html" >}}

{{< div-block-end >}}

{{< article-block-end >}}
