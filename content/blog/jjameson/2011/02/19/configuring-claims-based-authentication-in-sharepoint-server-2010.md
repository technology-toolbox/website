---
title: Configuring Claims-Based Authentication in SharePoint Server 2010
date: 2011-02-19T13:06:00-07:00
excerpt:
  "I thought it would be helpful to share my step-by-step procedures for
  manually configuring claims-based authentication in SharePoint Server 2010
  using an \"ASP.NET database\" and corresponding membership and role providers.
  Note that the following TechNet..."
aliases:
  [
    "/blog/jjameson/archive/2011/02/18/configuring-claims-based-authentication-in-sharepoint-server-2010.aspx",
    "/blog/jjameson/archive/2011/02/19/configuring-claims-based-authentication-in-sharepoint-server-2010.aspx",
  ]
draft: true
categories: ["My System", "SharePoint"]
tags: ["My System", "SharePoint 2010", "PowerShell"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2011/02/19/configuring-claims-based-authentication-in-sharepoint-server-2010.aspx"
---

I thought it would be helpful to share my step-by-step procedures for manually
configuring claims-based authentication in SharePoint Server 2010 using an
"ASP.NET database" and corresponding membership and role providers.

Note that the following TechNet article provides *some* of the steps for
configuring claims-based authentication in SharePoint Server 2010 (using the
LDAP provider instead of the ASP.NET SQL providers):

{{< reference
title="Configure forms-based authentication for a claims-based Web application (SharePoint Server 2010)"
linkHref="http://technet.microsoft.com/en-us/library/ee806890.aspx" >}}

I had originally intended this post to simply serve as a precursor to
[my next post](/blog/jjameson/2011/02/25/claims-login-web-part-for-sharepoint-server-2010),
but during the process of writing this post, I realized that there are many
pieces lacking from the TechNet article. For example, if you use the current
PowerShell script provided in the above TechNet article, you end up with a Web
application that doesn't support Search (because it does not enable both Windows
authentication as well as Forms-Based Authentication).

In this post, I'll share a "real world" process for creating and configuring a
Web application in SharePoint Server 2010 using claims-based authentication.

In the following procedures, assume that we are configuring the public Internet
site for Fabrikam Technologies (my favorite fictitious manufacturing company)
and we want to provide the ability for customers and partners to login and
access personalized content. User accounts for customers and partners are stored
in a SQL Server database (FabrikamDemo).

[User accounts for Fabrikam employees are stored in Active Directory.
Consequently, Fabrikam employees will not login using forms-based
authentication. Rather, in order to author content and manage the site, Fabrikam
employees authenticate with the site using Windows authentication (in other
words, via the generic login window that varies slightly depending on the Web
browser being used).]

The relevant service accounts for claims-based authentication are listed in the
following table.

{{< table class="small" caption="Table 1 - Service Accounts" >}}

| User Logon Name | Full Name | Description |
| --- | --- | --- |
| EXTRANET\svc-sharepoint | Service account for SharePoint | SharePoint farm account used to create and access the SharePoint configuration database. It also acts as the application pool identity account for the Central Administration site, as well as the application pool for the Security Token Service application. |
| EXTRANET\svc-web-fabrikam | Service account for Fabrikam Web site | Used for the application pool for the Fabrikam Web application |

{{< /table >}}

[Assume that Fabrikam has established an "extranet" Active Directory domain
which will be used to host the SharePoint farm. In order to allow Fabrikam
employees to authenticate with their internal domain (FABRIKAM) credentials, a
one-way trust is established from the EXTRANET domain to the FABRIKAM domain.]

Configuring claims-based authentication using a SQL Server database consists of
the following high-level steps:

1. Create and configure the membership/role database
2. Create the Web application and initial site collection (or configure an
   existing Web application to use claims-based authentication)
3. Configure SSL on the Web site
4. Enable anonymous access to the site
5. Modify the Web.config files for the following sites in order to support
   claims-based authentication:
   - SharePoint Central Administration v4
   - SecurityTokenServiceApplication
   - "Fabrikam" Web application
     ([http://www.fabrikam.com](http://www.fabrikam.com))
6. Create a user in the database using IIS Manager
7. Validate the configuration of the Web application

### Step 1 - Create and configure the membership/role database

In this step, the database for storing ASP.NET membership and role information
is created and the two service accounts specified in Table 1 are added to to the
appropriate database roles.

#### To create the database used for storing ASP.NET membership and role information:

1. Click **Start**, point to **All Programs**, click **Accessories**, and
   right-click **Command Prompt**, and then click **Run as administrator**.

2. At the command prompt, type the following command:
   
   ```
   cd %WinDir%\Microsoft.NET\Framework\v2.0.50727
   ```

3. Type the following command:
   
   ```
   aspnet_regsql.exe
   ```

4. On the welcome page of the **ASP.NET SQL Server Setup Wizard**, click
   **Next**.

5. On the **Select a Setup Option** page, ensure the option to **Configure SQL
   Server for application services** is selected and then click **Next**.

6. On the **Select the Server and Database** page:
   
   1. In the **Server** box, type the name of the database server.
   2. Ensure the **Windows authentication** option is selected.
   3. In the **Database** dropdown list, type **FabrikamDemo**.
   4. Click **Next**.

7. On the **Confirm Your Settings** page, verify the settings, and then click
   **Next**.

8. Wait for the database to be created and then click **Finish**.

#### To add the service accounts to the membership/role database:

1. On a computer with SQL Server management tools installed, click **Start**,
   point to **All Programs**, click **Microsoft SQL Server 2008**, and then
   click **SQL Server Management Studio**. The **Connect to Server** dialog box
   opens.
2. In the **Server type** list, click **Database Engine**.
3. Type the name of the server which hosts the database, and then click
   **Connect**.
4. In **Object Explorer**, expand **Security**, and then expand **Logins**.
5. Right-click the login corresponding to the SharePoint farm service account
   (**EXTRANET\svc-sharepoint**) and then click **Properties**.
6. In the login properties dialog box:
   1. On the **User Mapping** page, in the **Users mapped to the login** list,
      click the checkbox for the ASP.NET membership database (**FabrikamDemo**),
      and then in the database role membership list, click the checkboxes for
      the following roles:
      - **aspnet\_Membership\_BasicAccess**
      - **aspnet\_Membership\_ReportingAccess**
      - **aspnet\_Roles\_BasicAccess**
      - **aspnet\_Roles\_ReportingAccess**
   2. Click **OK**.
7. Repeat the steps in this section to add the service account for the Fabrikam
   Web application (**EXTRANET\svc-web-fabrikam**) to the following roles:
   - **aspnet\_Membership\_FullAccess**
   - **aspnet\_Roles\_BasicAccess**
   - **aspnet\_Roles\_ReportingAccess**

{{< div-block "note important" >}}

> **Important**
>
> Database access must be granted to both the service account used for the
> Fabrikam Web application and the SharePoint farm account. If the SharePoint
> farm account does not have access to the database, the Security Token Service
> used for claims-based authentication will be unable to validate the
> credentials.

{{< /div-block >}}

{{< div-block "note" >}}

> **Note**
>
> The reason the database roles are different between the two service accounts
> is because the SharePoint farm account only needs permissions to validate
> credentials and determine role membership, whereas the Fabrikam Web
> application service account needs additional permissions in order to support
> other scenarios for the Fabrikam site (e.g. "Change Password" and "Reset
> Password").

{{< /div-block >}}

### Step 2 - Create the Fabrikam Web application and initial site collection

In this step, the Web application and initial site collection are created.

#### To create the Fabrikam Web application:

1. On the **Start** menu, click **All Programs**, click **Microsoft SharePoint
   2010 Products**, right-click **SharePoint 2010 Management Shell**, and then
   click **Run as administrator**. If prompted by User Account Control to allow
   the program to make changes to the computer, click **Yes**.

2. From the Windows PowerShell command prompt, run the following script:
   
   ```
   $ErrorActionPreference = "Stop"
   
   $appPoolUserName = "EXTRANET\svc-web-fabrikam"
   $membershipProviderName = "FabrikamSqlMembershipProvider"
   $roleProviderName = "FabrikamSqlRoleProvider"
   $webAppName = "SharePoint - www.fabrikam.com80"
   $webAppUrl = "http://www.fabrikam.com"
   $contentDatabaseName = "WSS_Content_FabrikamDemo"
   $appPoolName = $webAppName
   
   Write-Debug "Get service account for application pool ($appPoolUserName)..."
   $appPoolAccount = Get-SPManagedAccount -Identity $appPoolUserName -EA 0
   
   if($appPoolAccount -eq $null)
   {
       Write-Host "Registering managed account ($appPoolUserName)..."
   
       Write-Debug "Get credential ($appPoolUserName)..."
       $appPoolCredential = Get-Credential $appPoolUserName
   
       $appPoolAccount = New-SPManagedAccount -Credential $appPoolCredential
   }
   
   $windowsAuthProvider = New-SPAuthenticationProvider
   $formsAuthProvider = New-SPAuthenticationProvider `
       -ASPNETMembershipProvider $membershipProviderName `
       -ASPNETRoleProviderName $roleProviderName
   
   $authProviders = $windowsAuthProvider, $formsAuthProvider
   
   $webApp = New-SPWebApplication -Name $webAppName -AllowAnonymousAccess `
       -ApplicationPool $appPoolName -AuthenticationMethod "NTLM" `
       -ApplicationPoolAccount $appPoolAccount -Url $webAppUrl -Port 80 `
       -AuthenticationProvider $authProviders -DatabaseName $contentDatabaseName
   ```

#### To create the initial site collection for the Web application:

1. If necessary, start an Administrator instance of the SharePoint 2010
   Management Shell.

2. From the Windows PowerShell command prompt, run the following script:
   
   ```
   $ErrorActionPreference = "Stop"
   
   $webAppUrl = "http://www.fabrikam.com"
   $siteName = "Fabrikam"
   $siteDescription = "Public Internet site for Fabrikam Technologies"
   $siteTemplate = "BLANKINTERNETCONTAINER#0"
   
   $ownerAlias = $env:USERDOMAIN + "\" + $env:USERNAME
   
   $siteUrl = $webAppUrl + "/"
   
   New-SPSite $siteUrl -OwnerAlias $ownerAlias -Name $siteName `
       -Description $siteDescription -Template $siteTemplate
   ```

### Step 3 - Configure SSL on the Web site

When using Forms-Based Authentication, it is important to secure the
communication between the clients and the Web servers (in order to avoid sending
user credentials in clear text over the network). In this section, the Web
application is modified to support both HTTP and HTTPS, and the corresponding
SSL certificate is configured for the Web site.

#### To add a public URL to HTTPS:

1. On the Central Administration home page, click **Application Management**.
2. On the **Application Management** page, in the **Web Applications** section,
   click **Configure alternate access mappings**.
3. On the **Alternate Access Mappings** page, click **Edit Public URLs**.
4. On the **Edit Public Zone URLs**page:
   1. In the **Alternate Access Mapping Collection** section, select the Web
      application to configure.
   2. In the **Public URLs** section, copy the URL from the **Default** box to
      the **Internet** box, and change **http://** to **https://**.
   3. Click **Save**.

#### To add an HTTPS binding to the site in IIS:

1. Click **Start**, point to **Administrative Tools**, and then click **Internet
   Information Services (IIS) Manager**.
2. In Internet Information Services (IIS) Manager, click the plus sign (+) next
   to the server name that contains the Web application, and then click the plus
   sign next to **Sites** to view the Web applications that have been created.
3. Click the name of the Web application (**SharePoint -- www.fabrikam.com80**).
   In the **Actions** section, under the **Edit Site** heading, click
   **Bindings...**.
4. In the **Site Bindings** window, click **Add**.
5. In the **Add Site Binding**window:
   1. In the **Type:** dropdown, select **https**.
   2. In the **SSL Certificate:** dropdown, select the certificate corresponding
      to the site.
   3. Click **OK**.
   4. In the **Site Bindings** window, click **Close**.

### Step 4 - Enable anonymous access to the site

In addition to enabling anonymous access on the Web application, the root Web of
the site collection must also be configured to enable anonymous access.

#### To enable anonymous access to the site:

1. If necessary, start an Administrator instance of the SharePoint 2010
   Management Shell.

2. From the Windows PowerShell command prompt, run the following script:
   
   ```
   $ErrorActionPreference = "Stop"
   
   Add-PSSnapin Microsoft.SharePoint.PowerShell -EA 0
   
   $webUrl = "http://www.fabrikam.com/"
   
   function EnableAnonymousAccess(
       [Microsoft.SharePoint.SPWeb] $web)
   {
       Write-Debug "Enabling anonymous access on site ($($web.Url))..."
   
       $anonymousPermissionMask =
           [Microsoft.SharePoint.SPBasePermissions]::Open `
           -bor [Microsoft.SharePoint.SPBasePermissions]::ViewFormPages `
           -bor [Microsoft.SharePoint.SPBasePermissions]::ViewListItems `
           -bor [Microsoft.SharePoint.SPBasePermissions]::ViewPages `
           -bor [Microsoft.SharePoint.SPBasePermissions]::ViewVersions
   
       if ($web.AnonymousPermMask64 -eq $anonymousPermissionMask)
       {
           Write-Debug `
               "Anonymous access is already enabled on site ($($web.Url))."
   
           return;
       }
   
       if ($web.HasUniqueRoleAssignments -eq $false)
       {
           $web.BreakRoleInheritance($true);
       }
   
       $web.AnonymousPermMask64 = $anonymousPermissionMask;
       $web.Update();
   
       Write-Host -Fore Green `
           "Successfully enabled anonymous access on site ($($web.Url))."
   }
   
   $DebugPreference = "SilentlyContinue"
   $web = Get-SPWeb $webUrl
   
   $DebugPreference = "Continue"
   EnableAnonymousAccess $web
   ```

### Step 5 - Add Web.config modifications for claims-based authentication

In order to complete the configuration of claims-based authentication, it is
necessary to modify the Web.config files for the following sites:

- SharePoint Central Administration v4
- Security Token Service
- The "Fabrikam" Web application (http://www.fabrikam.com)

#### To configure the Central Administration Web.config file:

1. Click **Start**, point to **Administrative Tools**, and then click **Internet
   Information Services (IIS) Manager**.

2. In **Internet Information Services (IIS) Manager**, in the **Connections**
   pane, click the plus sign (+) next to the server name that contains the Web
   application, and then click the plus sign next to **Sites** to view the Web
   applications that have been created.

3. Right-click **SharePoint Central Administration v4**, and then click **Explore**. Windows Explorer opens, with the directories for the selected Web application listed.
   
   {{< div-block "note important" >}}
   
   > **Important**
   > 
   > Before you make changes to the Web.config file, make a copy of it by using
   > a different name (for example, "Web - Copy.config"), so that if a mistake
   > is made in the file, you can delete it and use the original file.
   
   {{< /div-block >}}

4. Double-click the **Web.config** file to open the file.
   
   {{< div-block "note" >}}
   
   > **Note**
   > 
   > If you see a dialog box that says that Windows cannot open the file, click
   > **Select the program from a list**, and then click **OK**. In the **Open
   > With** dialog box, click **Notepad**, and then click **OK**.
   
   {{< /div-block >}}

5. In the Web.config editor:
   
   1. After the end of the **/configuration/configSections** element (i.e. `</configSections>`), add the following elements:
      
      ```
        <connectionStrings>
          <add name="FabrikamDemo"
            connectionString="Server={databaseServer};Database=FabrikamDemo;Integrated Security=true" />
        </connectionStrings>
      ```
      
      {{< div-block "note important" >}}
      
      > **Important**
      > 
      > Be sure to replace the **{databaseServer}** placeholder in the
      > connection string with the name of the database server.
      
      {{< /div-block >}}
   
   2. Find the **/configuration/system.web/roleManager/providers** section and add the following elements:
      
      ```
      <add name="FabrikamSqlRoleProvider"
        type="System.Web.Security.SqlRoleProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a"
        applicationName="Fabrikam Demo Site"
        connectionStringName="FabrikamDemo" />
      ```
   
   3. Find the **/configuration/system.web/membership/providers** section and add the following elements:
      
      ```
      <add name="FabrikamSqlMembershipProvider"
        type="System.Web.Security.SqlMembershipProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a"
        applicationName="Fabrikam Demo Site"
        connectionStringName="FabrikamDemo"
        passwordFormat="Hashed" />
      ```

6. Save the changes to the Web.config file and close the editor.

#### To configure the Security Token Service Web.config file:

1. In **Internet Information Services (IIS) Manager**, in the **Connections**
   pane, expand the **SharePoint Web Services** site, right-click the
   **SecurityTokenServiceApplication** subsite, and then click **Explore**.

2. Double-click the **Web.config** file to open the file.

3. In the Web.config editor, add the following elements to the `<configuration>` root element:
   
   ```
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

4. Save the changes to the Web.config file and close the editor.

#### To configure the Web.config file for the Fabrikam Web application:

1. In **Internet Information Services (IIS) Manager**, in the **Connections**
   pane, right-click the **SharePoint - www.fabrikam.com80** site, and then
   click **Explore**.
2. Double-click the **Web.config** file to open the file.
3. In the Web.config editor:
   1. After the end of the **/configuration/configSections** element (i.e. `</configSections>`), add the following elements:
      
      ```
        <connectionStrings>
          <add name="FabrikamDemo"
            connectionString="Server={databaseServer};Database=FabrikamDemo;Integrated Security=true" />
        </connectionStrings>
      ```
      
      {{< div-block "note important" >}}
      
      > **Important**
      > 
      > Be sure to replace the **{databaseServer}** placeholder in the
      > connection string with the name of the database server.
      
      {{< /div-block >}}
   
   2. Find the **/configuration/system.web/roleManager/providers** section and add the following elements:
      
      ```
      <add name="FabrikamSqlRoleProvider"
        type="System.Web.Security.SqlRoleProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a"
        applicationName="Fabrikam Demo Site"
        connectionStringName="FabrikamDemo" />
      ```
      
      {{< div-block "note warning" >}}
      
      > **Warning**
      > 
      > Do not overwrite any existing entries in this Web.config file.
      
      {{< /div-block >}}
   
   3. Find the **/configuration/system.web/membership/providers** section and add the following elements:
      
      ```
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
4. Save the changes to the Web.config file and close the editor.

### Step 6 - Create a user in the database using IIS Manager

#### To create a user for the Fabrikam Web site:

1. In **Internet Information Services (IIS) Manager**, click the Fabrikam Web
   application (e.g. **SharePoint -- www.fabrikam.com80**) and then double-click
   **.NET Users**.
2. When prompted with an error stating the feature cannot be used because the
   default provider is not a trusted provider, click **OK**.
3. In the **Actions** pane, click **Set Default Provider...**
4. In the **Edit .NET Users Settings** dialog box, note that the default
   provider configured in SharePoint Server 2010 is "i". In the **Default
   Provider** list, click **FabrikamSqlMembershipProvider**, and then click
   **OK**.
5. In the **Actions** pane, click **Add...**
6. When prompted with an error stating the default .NET Roles provider does not
   exist, click **OK**.
7. In the **Add .NET User**dialog:
   1. On the **.NET User Account Details** page, type the appropriate values in
      the **User Name**, **E-mail**, **Password**, **Confirm Password**,
      **Question**, and **Answer** boxes, and then click **Next**.
   2. On the **.NET User Roles** page, click **Finish**.
8. In the **Actions** pane, click **Set Default Provider...**
9. In the **Edit .NET Users Settings** dialog box, in the **Default Provider**
   list, click **i**, and then click **OK**.

### Step 7 - Validate the configuration of the Web application

The final step is to validate the Web application works as expected when using
both Forms-Based Authentication and Windows authentication.

#### To login to the Fabrikam Web site using Forms-Based Authentication:

1. Browse to the home page page the Fabrikam Web site (http://www.fabrikam.com)
   and click **Sign In**.
2. On the **Sign In**page:
   1. In the dropdown list, click **Forms Authentication**.
   2. When prompted to enter the **User name** and **Password**, type the
      credentials specified in the previous step and then click **Sign In**.
3. Verify the home page is displayed and the **Sign In** link has been replaced
   with the "Welcome" menu.

#### To login to the Fabrikam Web site using Windows authentication:

1. Add the Fabrikam Web site to the **Local intranet** zone (in order to seamlessly authenticate with the current domain credentials).
   
   {{< div-block "note" >}}
   
   > **Note**
   > 
   > This is discussed in more detail in the following blog post:
   > 
   > {{< reference title="Be \"In the Zone\" to Avoid Entering Credentials" linkHref="/blog/jjameson/2007/03/22/be-in-the-zone-to-avoid-entering-credentials" linkText="http://blogs.msdn.com/jjameson/archive/2007/03/22/be-in-the-zone-to-avoid-entering-credentials.aspx" >}}
   
   {{< /div-block >}}

2. Browse to the home page page the Fabrikam Web site (http://www.fabrikam.com)
   and click **Sign In**.

3. On the **Sign In** page, in the dropdown list, click **Windows
   Authentication**.

4. Verify the home page is displayed and the **Sign In** link has been replaced
   with the "Welcome" menu.

### What's next?

In
[my next post](/blog/jjameson/2011/02/25/claims-login-web-part-for-sharepoint-server-2010),
I explain how to create a custom Web Part that can be used to provide a
"branded" login page (instead of the generic "Sign In" page provided
out-of-the-box in SharePoint Server 2010).
