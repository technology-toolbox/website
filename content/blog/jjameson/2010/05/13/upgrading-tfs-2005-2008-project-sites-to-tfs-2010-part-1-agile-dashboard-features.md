---
title: "Upgrading TFS 2005/2008 Project Sites to TFS 2010, Part 1 - Agile Dashboard Features"
date: 2010-05-13T21:57:00-07:00
excerpt: "In one of last week's posts , I provided details on upgrading from Team Foundation Server 2008 to TFS 2010, including some information about updating your TFS project sites. I also provided a reference to the following MSDN article for more information..."
draft: true
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "TFS", "SharePoint 
			2010", "PowerShell"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/05/14/upgrading-tfs-2005-2008-project-sites-to-tfs-2010-part-1-agile-dashboard-features.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/05/14/upgrading-tfs-2005-2008-project-sites-to-tfs-2010-part-1-agile-dashboard-features.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that
> blog ever goes away.

In
[one of last week's posts](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010), I provided details on upgrading from Team Foundation
Server 2008 to TFS 2010, including some information about updating your TFS
project sites. I also provided a reference to the following MSDN article for
more information:

{{< reference title="Updating an Upgraded Team Project to Access New Features" linkHref="http://msdn.microsoft.com/en-us/library/ff432837.aspx" >}}

By following the above MSDN documentation, I successfully added a Product
Backlog workbook as well as an Iteration Backlog workbook. [Note that I used
[Hakan Eskici's script to automate the rather tedious process of updating the
work item types](http://blogs.msdn.com/hakane/archive/2010/04/27/sample-script-to-enable-new-features-in-upgraded-team-projects-tfs-2010-rtm.aspx) to support the workbook queries -- for example to add the
**Story Points** field to **Scenario** work items.]

Logically, my next step was to add the new TFS 2010 dashboard functionality
to my existing project sites.

The steps to do this are documented in the following MSDN article (which
is part of the documentation set referenced above):

{{< reference title="Adding Dashboards and Reports to Upgraded Team Projects" linkHref="http://msdn.microsoft.com/en-us/library/ff462695(v=VS.100).aspx" >}}

However, there's a problem with the steps described in this article...

...they simply don't work.

Here is the XML input file that I used with the **File.BatchNewTeamProject**
command:

```
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="ProjectCreationSettingsFileSchema.xsd">
  <TFSName>http://cyclops:8080/tfs/DefaultCollection</TFSName>
  <LogFolder>C:\NotBackedUp\Temp</LogFolder>
  <ProjectName>AdventureWorks</ProjectName>
  <AddFeaturesToExistingProject>true</AddFeaturesToExistingProject>
  <ProjectReportsEnabled>false</ProjectReportsEnabled>
  <ProjectSiteEnabled>true</ProjectSiteEnabled>
  <ProjectSiteWebApplication>http://cyclops</ProjectSiteWebApplication>
  <ProjectSitePath>sites/AdventureWorks</ProjectSitePath>
  <ProjectSiteTitle>Adventure Works</ProjectSiteTitle>
  <!--
  <ProjectSiteDescription>This team project was created based on the 'MSF for Agile Software Development - v4.2' process template, but subsequently upgraded for TFS 2010.</ProjectSiteDescription>
  -->
  <ProjectSiteDescription></ProjectSiteDescription>
  <ProcessTemplateName>MSF for Agile Software Development v5.0</ProcessTemplateName>
</Project>
```

> **Note**
>
> You must run the **File.BatchNewTeamProject** command
> directly on the TFS/SharePoint Server (which means you must have previously
> installed Team Explorer or Visual Studio on the server) -- not from
> a remote client with Visual Studio.
>
> If you attempt to run it from a remote client, you'll get an error
> that suggests you specified an invalid value for the SharePoint Web
> application (i.e. the `<ProjectSiteWebApplication>`element). Apparently, this command ultimately invokes
> the SharePoint object model and therefore cannot find the SharePoint
> Web application if you try to run it remotely.

After running the command on the TFS/SharePoint Server with the above input
file, I noticed that it didn't run for very long (i.e. the word "Ready" appeared
almost immediately in the status bar in Visual Studio). Looking at the corresponding
AdventureWorks.log file, I discovered the following:

{{< blockquote "font-italic text-danger" >}}

Exception Message: TF30270: The following team project site cannot be created because it already exists: AdventureWorks. Either specify another location for the team project site, or specify a different name for the team project.

{{< /blockquote >}}

Well, yeah, of course the team project site already exists -- that's why
I'm trying to upgrade it with the new TFS 2010 features! Isn't that why I specified
the following in my input file?

`<AddFeaturesToExistingProject>true</AddFeaturesToExistingProject>`

That's when I noticed
[justbail](http://msdn.microsoft.com/en-us/library/community/user/81794.aspx)'s comment on the above MSDN article, in which he -- or she -- suggested
the workaround is to simply create a new project site by specifying a different
site URL (which essentially means you have your "legacy" project site with content
such as shared documents and list items, and a new project site with your project
dashboards). While that alternative might be acceptable for some people, it
definitely left me feeling a little queasy.

Since I was essentially blocked at that point, I decided to try running the
command again, after making the following change to my input file:

`<ProjectSiteEnabled>false</ProjectSiteEnabled>`

I noticed that the process ran for considerably longer (i.e. it took several
seconds before the word "Ready" appeared in the status bar) so I was really
hoping -- okay, actually *expecting* -- to see some dashboards when I
browsed to my project site...

...but, no, they weren't there after all.

I looked at the log file again and, sure enough, a bunch of activity actually
took place. However, while the log file indicated success ("Team Project Batch
Creation succeeded"), there definitely were no dashboards to be found on my
upgraded project site.

At that point, I decided to take a different approach altogether.

If you look at the site features for a new project site created with the
MSF Agile v5 template TFS 2010 (click **Site Settings**, then in the **Site Actions** section, click **Manage
site features**), you will notice the following features are activated:

**Site (Web) Features**

| Display Name  | Description  | Name  | Id  |
| --- | --- | --- | --- |
| Agile Dashboards  | Activate if the team project does not have reporting enabled and
the project was created using the MSF for Agile Software Development
v5.0 process template.  | TfsDashboardAgileNoWh  | f25ef169-2fe5-4717-9ba3-7dc1ecd6e514  |
| Agile Dashboards with Basic Reporting  | Activate if the team project has reporting enabled, but you do not
have Excel Services enabled to render Excel reports. Activate this feature
for projects created using the MSF for Agile Software Development v5.0
process template.  | TfsDashboardAgileWss  | ced2ceba-43ac-4535-946a-70605e721d37  |
| Agile Dashboards with Excel Reporting  | Activate if the team project has reporting enabled and is using
a supported edition of Microsoft Office SharePoint Server 2007 or Microsoft
SharePoint Server 2010 with Excel Services. Activate this feature for
projects created using the MSF for Agile Software Development v5.0 process
template.  | TfsDashboardAgileMoss  | 0d953ee4-b77d-485b-a43c-f5fbb9367207  |
| Team Collaboration Lists  | Provides team collaboration capabilities for a site by making standard
lists, such as document libraries and issues, available.  | TeamCollab  | 00bfea71-4ea5-48d4-a4ad-7ea5c011abe5  |

Note that there are other activated features (e.g. **Offline Synchronization
for External Lists**), but from a TFS perspective -- and the purposes
of this post -- only the four features listed above are of interest.

Similarly, if you look at the corresponding site collection features (click
**Site Settings**, then in the **Site Collection Administration** section, click **Go to top level site settings**, then
click **Site collection features**), you will notice the following
feature is activated:

**Site Collection Features**

| Display Name  | Description  | Name  | Id  |
| --- | --- | --- | --- |
| Visual Studio Team Foundation Server Web Part Collection  | Collection of web parts to display various information from a Team
Foundation Server instance.  | TswaWebParts  | cce226d2-d7b9-44fb-b5be-a1ccf91cbd90  |

After poking around a little bit in the SharePoint feature files installed
by TFS 2010 and looking at the corresponding feature assembly using Reflector,
it quickly became apparent that the TFS product team approaches SharePoint development
in much the same way that I do (i.e. by activating features on a site to install
and configure new functionality). It's something I like to refer to as
[the "DR.DADA" approach to SharePoint](/blog/jjameson/2009/03/31/introducing-the-dr-dada-approach-to-sharepoint-development) -- as opposed to
[the SharePoint
guidance on MSDN](http://msdn.microsoft.com/en-us/library/dd203468.aspx), which suggests using things like custom site definitions
(shudder) to implement your customizations.

If you take a quick look at the feature activation code, you'll see that
the dashboard features incrementally install a list of features (e.g. TeamCollab,
TswaWebParts, TfsDashboardContentTypes, ...) until you get the level of functionality
you desire (for example, minimal dashboard functionality or the full blown dashboards
with Excel reporting -- assuming you are running Microsft Office SharePoint
Server (MOSS) 2007 or SharePoint Server 2010).

Based on my investigation, I decided to activate the **Agile Dashboards
with Excel Reporting** feature on my site.

Note that you can activate the feature in a variety of ways:

- On the **Features** page in **Site Settings** (e.g.
  [http://cyclops/sites/AdventureWorks/\_layouts/ManageFeatures.aspx](http://cyclops/sites/AdventureWorks/_layouts/ManageFeatures.aspx))
- Using StsAdm.exe (e.g. {{< kbd "stsadm -o activatefeature -name TfsDashboardAgileMoss -url http://cyclops/sites/AdventureWorks" >}})
- If you are running SharePoint Server 2010, using PowerShell and the
  **[Enable-SPFeature](http://technet.microsoft.com/en-us/library/ff607803%28office.14%29.aspx)** cmdlet.

Since I have a number of TFS project sites to upgrade, I chose to activate
the feature using PowerShell:

```
Enable-SPFeature "TfsDashboardAgileMoss" -Url "http://cyclops/sites/AdventureWorks"
```

After the feature finished activating, I once again browsed to my project
site -- and this time was quite happy to find the dashboards had been created.

> **Important**
>
> Even though the dashboards have been created, you still have some
> work to do (assuming you want the dashboards and new TFS Web Parts to
> show accurate information). This is due to the fact that the **Scenario** work item type was renamed to **User Story**
> in MSF Agile v5, and consequently, the queries specified in some of
> the dashboard Web Parts and Excel reports need to be updated accordingly.
>
> I'll cover these configuration changes in a separate post.

Note that when you create a new project site in TFS 2010 using the MSF Agile
v5 template, the Burndown dashboard is set as the default page for the site.

To make the Burndown dashboard the default page for an upgraded project site:

1. Browse to the home page of the project site (e.g.
   [http://cyclops/sites/AdventureWorks](http://cyclops/sites/AdventureWorks)).
2. In the quick launch navigation on the left, under the **Dashboards** heading, click **Burndown**.
3. On the **Burndown** page, select the **Page** tab.
4. In the **Page Actions** group of the Ribbon, click
   **Make Homepage**. When prompted to set the page as the site's
   home page, click **OK**.

If you want your upgraded project site to have similar navigation to a new
project site created with the MSF Agile v5 template, you can configure the quick
launch navigation accordingly.

Lastly, note that if you use PowerShell to activate the dashboard feature,
you can easily upgrade numerous project sites at once:

```
$sitesToUpgrade =
   @(
"http://cyclops/sites/AdventureWorks",
"http://cyclops/sites/Demo",
"http://cyclops/sites/Toolbox"
   )

$sitesToUpgrade |
   ForEach-Object {
      Enable-SPFeature "TfsDashboardAgileMoss" -Url $_
   }
```

