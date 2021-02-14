---
title: "Upgrading TFS 2005/2008 Project Sites to TFS 2010, Part 4 - Project Work Items Web Part"
date: 2010-05-17T09:31:00+08:00
excerpt: "In the part 1 of this series , I described how to enable the dashboard functionality in Team Foundation Server (TFS) 2010 on project sites upgraded from TFS 2005/2008 (i.e. sites originally created with the MSF Agile v4.x process templates). I noted that..."
draft: true
categories: ["Development", "SharePoint"]
tags: ["TFS", "SharePoint 2010", "PowerShell"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2010/05/17/upgrading-tfs-2005-2008-project-sites-to-tfs-2010-part-4-project-work-items-web-part.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/05/17/upgrading-tfs-2005-2008-project-sites-to-tfs-2010-part-4-project-work-items-web-part.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

In the [part 1 of this series](/blog/jjameson/2010/05/14/upgrading-tfs-2005-2008-project-sites-to-tfs-2010-part-1-agile-dashboard-features), I described how to enable the dashboard functionality in Team Foundation Server (TFS) 2010 on project sites upgraded from TFS 2005/2008 (i.e. sites originally created with the MSF Agile v4.x process templates). I noted that it wasn't quite as simple as activating a feature on the SharePoint site because some of the TFS Web Parts on the new dashboards specify the new **User Story** work item type introduced in the MSF Agile v5 process template, instead of the old **Scenario** work item type.

[Honestly, I'm not sure why the TFS team felt compelled to change the name of the work items. My preference would have been to keep the original work item type and extend it as necessary (e.g. to add a "Story Points" field) -- if for no other reason than to avoid having to update numerous Web Parts and Excel reports on upgraded TFS project sites. Obviously the TFS product group didn't ask for my opinion on the matter ;-)

Personally, I have yet to meet anyone who can't understand that a *scenario* is essentially the same as a *user story*. Then again, I still like to use *interim milestones* (e.g. M0, M1, M2, ...) rather than *iterations* (Iteration 0, Iteration 1, Iteration 2, ...), so I guess I'm just "old school" in some regards.]

Assuming you want to keep using your existing TFS project (with its **Scenario** work item type) but also leverage the great new dashboard features in TFS 2010, then you need to update the query criteria specified by the TFS Web Parts and Excel workbooks. In this post, I'll provide you with a short PowerShell script that makes it really easy to upgrade the numerous Web Parts that need to be updated on the various dashboard pages.

Let's start with the Burndown dashboard (which hopefully you'll set as the default page for the upgraded project site, since it provides an excellent summary view of the overall health of your project).

On the right side of the Burndown dashboard, you'll find the **Project Work Items** Web Part. This is simply an instance of the **Microsoft.TeamFoundation.WebAccess.WebParts.WorkItemSummaryWebPart** with the **Query** property set to the following:

```
SELECT [System.Id], [System.Title]
FROM WorkItems
WHERE
(
    [System.TeamProject] = @project
    AND
    (
        [System.WorkItemType] = 'Bug'
        OR [System.WorkItemType] = 'Task'
        OR [System.WorkItemType] = 'Test Case'
        OR [System.WorkItemType] = 'User Story'
    )
)
ORDER BY [System.Id]
```

See the problem with this Web Part and upgraded MSF Agile v4 TFS projects?

Too bad the query specified for the Web Part doesn't explicitly *exclude* "unwanted" work item types (i.e. **Issue** and **Shared Steps**) instead of *including* a list of specific work item types. In other words, if the Web Part specified the following query, then **Scenario** work items would automatically be shown on upgraded project sites:

```
SELECT [System.Id], [System.Title]
FROM WorkItems
WHERE
(
    [System.TeamProject] = @project
    AND
    (
        [System.WorkItemType] <> 'Issue'OR [System.WorkItemType] <> 'Shared Steps'
    )
)
ORDER BY [System.Id]
```

Consequently, for upgraded project sites we need to update the Web Part query to include **Scenario** work items instead of **User Story** work items. I suggest applying a very minimal change to the Web Part query in order to specify the following:

```
SELECT [System.Id], [System.Title]
FROM WorkItems
WHERE
(
    [System.TeamProject] = @project
    AND
    (
        [System.WorkItemType] = 'Bug'
        OR [System.WorkItemType] = 'Task'
        OR [System.WorkItemType] = 'Test Case'
        OR [System.WorkItemType] = 'Scenario'
    )
)
ORDER BY [System.Id]
```

If you've done much SharePoint development, or if you've seen my [previous post on the **SharePointWebPartManager** class](/blog/jjameson/2009/10/17/introducing-the-sharepointwebparthelper-class), then you are likely familiar with programmatically manipulating Web Parts using the **[SPLimitedWebPartManager](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.webpartpages.splimitedwebpartmanager.aspx)** class.

Before listing the PowerShell script that will update the query on the **Project Work Items** Web Part, it is important to note that the Web Part is actually displayed on each of the dashboard pages:

- Burndown
- Quality
- Bugs
- Test
- Build
- My Dashboard

Since each instance of the Web Part specifies its own query, we need to update the Web Part on each of the pages:

```
# Updates the "Project Work Items" Web Part on the dashboard pages of TFS
# project sites originally created with the MSF Agile v4.x process template
# in order to show "Scenario" work items (instead of "User Story" work items)

function UpdateWorkItemSummaryWebPartOnEachDashboardPage(
    [Microsoft.SharePoint.SPWeb] $web)
{
    Write-Debug "Updating work item summary Web Parts on site ($($web.Url))..."
    
    $pagesToUpdate =
    @(
        "Dashboards/Burndown.aspx",
        "Dashboards/Quality.aspx",
        "Dashboards/Bugs.aspx",
        "Dashboards/Test.aspx",
        "Dashboards/Build.aspx",
        "Dashboards/MyDashboard.aspx"
    )
    
    $pagesToUpdate |
        ForEach-Object {                        
            $wpm = $web.GetLimitedWebPartManager(
                $_,
                [System.Web.UI.WebControls.WebParts.PersonalizationScope]::Shared)
            
            UpdateWorkItemSummaryWebParts $wpm
            $wpm.Dispose()
        }
}

function UpdateWorkItemSummaryWebParts(
    [Microsoft.SharePoint.WebPartPages.SPLimitedWebPartManager] $wpm)
{
    Write-Debug "Updating work item summary Web Part on page ($($wpm.ServerRelativeUrl))..."
    
    $wpm.WebParts |
        Where-Object {
            $_ -is [Microsoft.TeamFoundation.WebAccess.WebParts.WorkItemSummaryWebPart]} |
        ForEach-Object {
            UpdateWorkItemSummaryWebPart $wpm $_      
        }
}

function UpdateWorkItemSummaryWebPart(
    [Microsoft.SharePoint.WebPartPages.SPLimitedWebPartManager] $wpm,
    [Microsoft.TeamFoundation.WebAccess.WebParts.WorkItemSummaryWebPart] $webPart)
{
    Write-Debug "Updating Web Part ($($webPart.Title))..."

    $webPart.Query = $webPart.Query.Replace("User Story", "Scenario")
    $wpm.SaveChanges($webPart)
    
    Write-Debug "Successfully updated Web Part ($($webPart.Title))."
}

$sitesToUpgrade =
    @(
        "http://cyclops/sites/AdventureWorks",
        "http://cyclops/sites/Demo",
        "http://cyclops/sites/Toolbox"
    )

$sitesToUpgrade |
    ForEach-Object {
        $DebugPreference = "SilentlyContinue"
        $web = Get-SPWeb $_

        $DebugPreference = "Continue"
        UpdateWorkItemSummaryWebPartOnEachDashboardPage $web

        $web.Dispose()
    }
```

After running this script, the **Project Work Items** Web Part on each dashboard page will show scenarios in addition to the other work item types in the MSF Agile v4 process template.

