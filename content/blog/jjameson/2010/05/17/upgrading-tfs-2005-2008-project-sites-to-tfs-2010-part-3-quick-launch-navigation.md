---
title: Upgrading TFS 2005/2008 Project Sites to TFS 2010, Part 3 - Quick Launch Navigation
date: 2010-05-17T18:49:00-06:00
excerpt:
  Update (2010-05-20) I made some changes to correct a few issues and also to
  include the final version of the XML input file that I used to update my TFS
  project sites. In my previous post , I showed how you can use PowerShell to
  export the quick...
aliases:
  [
    "/blog/jjameson/archive/2010/05/17/upgrading-tfs-2005-2008-project-sites-to-tfs-2010-part-3-quick-launch-navigation.aspx",
  ]
draft: true
categories: ["Development", "SharePoint"]
tags: ["TFS", "SharePoint 2010", "PowerShell"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/05/17/upgrading-tfs-2005-2008-project-sites-to-tfs-2010-part-3-quick-launch-navigation.aspx"
---

{{< div-block "note" >}}

> **Update (2010-05-20)**
>
> I made some changes to correct a few issues and also to include the final
> version of the XML input file that I used to update my TFS project sites.

{{< /div-block >}}

In my
[previous post](/blog/jjameson/2010/05/17/configuring-quick-launch-navigation-in-sharepoint-server-2010-using-powershell),
I showed how you can use PowerShell to export the quick launch navigation from a
SharePoint site to XML and subsequently add, update, and reorder navigation
nodes by modifying the XML accordingly. In this post, I'll demonstrate a
practical application of that process.

Project sites created using the MSF Agile v4.2 process template in Team
Foundation Server (TFS) 2008 have the following quick launch navigation:

- Documents
  - Development
  - Project Management
  - Requirements
  - Security
  - Test
- Process Guidance
- Reports
  - Bug Rates
  - Builds
  - Quality Indicators
  - Project Velocity
  - Issues List
  - Exit Criteria Status

Assuming you are using Microsoft Office SharePoint Server (MOSS) 2007 or
SharePoint Server 2010 (in order to leverage Excel Services for the dashboard
functionality), project sites created using the MSF Agile v5 process template in
TFS 2010 have the following quick launch navigation:

- Team Web Access
- Dashboards
  - Burndown
  - Quality
  - Bugs
  - Test
  - Build
  - My Dashboard
- Excel Reports
- Reports
- Libraries
  - Team Wiki
  - Shared Documents
  - Samples and Templates
- Lists
  - Calendar
- Process Guidance

Therefore if you want to update the quick launch navigation on project sites
upgraded from TFS 2008 (or TFS 2005) -- and you don't want to make the
configuration changes manually -- then start by exporting the quick launch
navigation from a new project site created in TFS 2010 (for example,
[http://cyclops/sites/DefaultCollection/Test](http://cyclops/sites/DefaultCollection/Test)):

```
# Exports the quick launch navigation for a SharePoint site as XML

function ExportQuickLaunchNavigation(
    [Microsoft.SharePoint.SPWeb] $web)
{
    $xml = [xml] "<QuickLaunch/>"

    foreach ($navigationNode in $web.Navigation.QuickLaunch)
    {
        AddNavigationElement $navigationNode $xml.DocumentElement $web
    }

    return $xml
}

function AddNavigationElement(
    [Microsoft.SharePoint.Navigation.SPNavigationNode] $navigationNode,
    [System.Xml.XmlElement] $parentElement,
    [Microsoft.SharePoint.SPWeb] $web)
{
    $url = $navigationNode.Url.Replace(
        $web.ServerRelativeUrl,
        "(`$web.ServerRelativeUrl)")

    $navElement = $parentElement.OwnerDocument.CreateElement("NavigationNode")

    $parentElement.AppendChild($navElement) > $null

    $navElement.SetAttribute("title", $navigationNode.Title)
    $navElement.SetAttribute("url", $url)

    foreach ($childNode in $navigationNode.Children)
    {
        AddNavigationElement $childNode $navElement $web
    }
}

$web = Get-SPWeb "http://cyclops/sites/DefaultCollection/Test"

$navigationXml = ExportQuickLaunchNavigation($web)

$navigationXml.OuterXml > QuickLaunch-TFS2010.xml
```

This will output XML similar to the following:

```
<QuickLaunch>
  <NavigationNode
    title="Team Web Access"
    url="($web.ServerRelativeUrl)/_layouts/tfsredirect.aspx?tf%3aType=WebAccess" />
  <NavigationNode
    title="Dashboards"
    url="($web.ServerRelativeUrl)/Dashboards/Forms/AllItems.aspx">
    <NavigationNode
      title="Burndown"
      url="($web.ServerRelativeUrl)/Dashboards/Burndown.aspx" />
    <NavigationNode
      title="Quality"
      url="($web.ServerRelativeUrl)/Dashboards/Quality.aspx" />
    <NavigationNode
      title="Bugs"
      url="($web.ServerRelativeUrl)/Dashboards/Bugs.aspx" />
    <NavigationNode
      title="Test"
      url="($web.ServerRelativeUrl)/Dashboards/Test.aspx" />
    <NavigationNode
      title="Build"
      url="($web.ServerRelativeUrl)/Dashboards/Build.aspx" />
    <NavigationNode
      title="My Dashboard"
      url="($web.ServerRelativeUrl)/Dashboards/MyDashboard.aspx" />
  </NavigationNode>
  <NavigationNode
    title="Excel Reports"
    url="($web.ServerRelativeUrl)/Reports/Forms/AllItems.aspx" />
  <NavigationNode
    title="Reports"
    url="($web.ServerRelativeUrl)/_layouts/tfsredirect.aspx?tf%3aType=ReportList" />
  <NavigationNode
    title="Libraries"
    url="($web.ServerRelativeUrl)/_layouts/viewlsts.aspx?BaseType=1">
    <NavigationNode
      title="Team Wiki"
      url="($web.ServerRelativeUrl)/Team Wiki" />
    <NavigationNode
      title="Shared Documents"
      url="($web.ServerRelativeUrl)/Shared Documents/Forms/AllItems.aspx" />
    <NavigationNode
      title="Samples and Templates"
      url="($web.ServerRelativeUrl)/Samples and Templates/Forms/AllItems.aspx" />
  </NavigationNode>
  <NavigationNode
    title="Lists"
    url="($web.ServerRelativeUrl)/_layouts/viewlsts.aspx?BaseType=0">
    <NavigationNode
      title="Calendar"
      url="($web.ServerRelativeUrl)/Lists/Calendar/calendar.aspx" />
  </NavigationNode>
  <NavigationNode
    title="Process Guidance"
    url="($web.ServerRelativeUrl)/_layouts/tfsredirect.aspx?tf%3aType=ProcessGuidance&amp;tf%3aDocumentPath=Supporting+Files%2fProcessGuidance.htm" />
</QuickLaunch>
```

Notice that the PowerShell script from my previous post has been updated to
automatically tokenize the URL for each navigation node (i.e. replacing the
server relative URL with "`($web.ServerRelativeUrl)`"). This token is
subsequently replaced with the new server relative URL when importing the quick
launch navigation.

Before we can import the navigation nodes, we need to tweak the XML slightly to
account for the fact that in TFS 2008 project sites, the **Event** list is
typically used for team calendar items (not a list named **Calendar**). This
assumes, of course, that you didn't customize the team site.

However, for consistency with new TFS 2010 project sites (in terms of
navigation), we'll keep the title of the quick launch navigation node as
"Calendar" and simply refer to the **Calendar** view of the **Event** list:

```
    <NavigationNode
      title="Calendar"
      url="($web.ServerRelativeUrl)/Lists/Events/calendar.aspx" />
```

We also need to add nodes for the existing document libraries (e.g.
**Development**) under the new **Libraries** heading:

```
    <NavigationNode
      title="Development"
      url="($web.ServerRelativeUrl)/Development/Forms/AllItems.aspx" />
    <NavigationNode
      title="Project Management"
      url="($web.ServerRelativeUrl)/Project%20Management/Forms/AllItems.aspx" />
    <NavigationNode
      title="Requirements"
      url="($web.ServerRelativeUrl)/Requirements" />
    <NavigationNode
      title="Security"
      url="($web.ServerRelativeUrl)/Security" />
    <NavigationNode
      title="Test"
      url="($web.ServerRelativeUrl)/Test/Forms/AllItems.aspx" />
```

These new navigation nodes compensate for the fact the **Documents** heading is
removed when updating the Quick Launch navigation. [This is removed because the
link (e.g. /sites/AdventureWorks/\_layouts/1033/viewlsts.aspx?BaseType=1 ) is no
longer valid -- due to the removal of the LCID (e.g. 1033) from the URL.]

Also note that project sites created with TFS 2010 do not include the process
guidance directly on the site (unlike project sites created in TFS 2005/2008).
Rather, the Process Guidance link now refers to an HTML page that simply
redirects to a page on the Microsoft site (e.g.
[http://go.microsoft.com/fwlink/?LinkId=153652&clcid=0x409](http://go.microsoft.com/fwlink/?LinkId=153652&clcid=0x409)).
Consequently, if you were to simply import the XML above at this point, you
would end up with two **Process Guidance** links in the quick launch navigation
(the first referring to the new HTML redirect page, and the second referring to
the deprecated process guidance that resides within the project site).

Fortunately, this is easy to remedy with a little more PowerShell script.
Suppose we want to remove the **Process Guidance** navigation link from a TFS
project site originally created in TFS 2008 (e.g.
[http://cyclops/sites/AdventureWorks](http://cyclops/sites/AdventureWorks)):

```
# Deletes a quick launch navigation link from a SharePoint site

function DeleteNavigationNode(
    [Microsoft.SharePoint.Navigation.SPNavigationNodeCollection] $nodes,
    [string] $titleOrUrl)
{
    $node = $nodes | Where-Object {
        ($_.Title -eq $titleOrUrl) -or ($_.Url -eq $titleOrUrl)}

    If ($node -ne $null)
    {
        If ($node -is [System.Object[]])
        {
            foreach ($tmpNode in $nodes)
            {
                Write-Debug ("Deleting navigation node" `
                    + " ($($tmpNode.Title) - $($tmpNode.Url))...")

                $tmpNode.Delete()
            }
        }
        Else
        {
            Write-Debug ("Deleting navigation node" `
                + " ($($node.Title) - $($node.Url))...")

            $node.Delete()
        }
    }
    Else
    {
        # No node found with specified title or URL (recurse on child nodes)
        foreach ($node in $nodes)
        {
            If ($node.Children.Count -gt 0)
            {
                DeleteNavigationNode $node.Children $titleOrUrl
            }
        }
    }
}

$DebugPreference = "SilentlyContinue"
$web = Get-SPWeb "http://cyclops/sites/AdventureWorks"

$DebugPreference = "Continue"
DeleteNavigationNode $web.Navigation.QuickLaunch "Process Guidance"
```

{{< div-block "note important" >}}

> **Important**
>
> After a navigation node has been deleted, you need to refresh the SPWeb object
> to avoid errors like the following:
>
> {{< blockquote "fst-italic text-danger" >}}
>
> An error occurred while enumerating through a collection: Cannot complete this
> action.
>
> {{< /blockquote >}}
>
> To refresh the SPWeb object (and consequently the associated
> SPNavigationNodeCollection), simply call the `Get-SPWeb` cmdlet again:
>
> `$web = Get-SPWeb "http://cyclops/sites/AdventureWorks"`

{{< /div-block >}}

Now we can import the quick launch navigation from the XML into the project
site:

```
# Imports the quick launch navigation for a SharePoint site from the specified
# XML (adding, renaming, and moving navigation nodes as necessary)

function EnsureNavigationNode(
    [Microsoft.SharePoint.Navigation.SPNavigationNodeCollection] $nodes,
    [string] $title,
    [string] $url)
{
    Write-Debug "Ensuring navigation node ($title - $url)..."

    [Microsoft.SharePoint.Navigation.SPNavigationNode] $node =
        $nodes | Where-Object {$_.Url -eq $url}

    If ($node -eq $null)
    {
        Write-Debug "Creating new navigation node ($title - $url)..."

        $node = New-Object Microsoft.SharePoint.Navigation.SPNavigationNode(
            $title,
            $url,
            $true)

        $null = $nodes.AddAsLast($node)
        $nodes.Navigation.Web.Update()
    }

    If ($node.Title -ne $title)
    {
        Write-Debug ("Updating title of navigation node ($($node.Title)) to" `
            + "($title)...")

        $node.Title = $title
        $node.Update()
    }

    return $node
}

function ImportNavigationNodes(
    [Microsoft.SharePoint.Navigation.SPNavigationNodeCollection] $nodes,
    [System.Xml.XmlNodeList] $navElements,
    [Microsoft.SharePoint.SPWeb] $web)
{
    [int] $position = 0

    foreach ($navElement in $navElements)
    {
        $title = $navElement.GetAttribute("title")
        $url = $navElement.GetAttribute("url")

        $url = $url.Replace(
            "(`$web.ServerRelativeUrl)",
            $web.ServerRelativeUrl)

        [Microsoft.SharePoint.Navigation.SPNavigationNode] $navigationNode =
            EnsureNavigationNode $nodes $title $url

        If ($position -eq 0)
        {
            $navigationNode.MoveToFirst($nodes)
        }
        Else
        {
            $navigationNode.Move($nodes, $nodes[$position - 1])
        }

        $position = $position + 1

        $childNodes = $navElement.SelectNodes("NavigationNode")
        If ($childNodes.Count -gt 0)
        {
            ImportNavigationNodes $navigationNode.Children $childNodes $web
        }
    }
}

function ImportQuickLaunchNavigation(
    [Microsoft.SharePoint.SPWeb] $web,
    [xml] $navigationXml)
{
    Write-Debug "Importing quick launch navigation for site ($($web.Url))..."

    $nodes = $web.Navigation.QuickLaunch

    $navElements = $navigationXml.SelectNodes(
        "/QuickLaunch/NavigationNode")

    If ($navElements.Count -gt 0)
    {
        ImportNavigationNodes $nodes $navElements $web
    }
    Else
    {
        Write-Host "No navigation nodes found to import."
    }
}

[xml] $navigationXml = Get-Content .\QuickLaunch-TFS2010.xml

$DebugPreference = "SilentlyContinue"
$web = Get-SPWeb "http://cyclops/sites/AdventureWorks"

$DebugPreference = "Continue"
ImportQuickLaunchNavigation $web $navigationXml
```

After running this script, the upgraded project site should contain the
following quick launch navigation:

- Team Web Access
- Dashboards
  - Burndown
  - Quality
  - Bugs
  - Test
  - Build
  - My Dashboard
- Excel Reports
- Reports
- Libraries
  - Team Wiki
  - Shared Documents
  - Samples and Templates
  - Development
  - Project Management
  - Requirements
  - Security
  - Test
- Lists
  - Calendar
- Process Guidance

Note that the **Libraries** heading (which was previously labeled **Documents**)
includes links to the original document libraries (e.g. **Development**,
**Project Management**, etc.) as well as links to new libraries used with the
MSF Agile v5 template (e.g. **Shared Documents** and **Sample and Templates**)
that don't currently exist in the upgraded site. With a little more work, you
could create the new libraries as needed. However, creating a new **Shared
Documents** library could potentially confuse team members when deciding where
to upload new documents.

Consequently, I chose to simply remove the links for **Shared Documents** and
**Samples and Templates** from the quick launch navigation.

If you want to upgrade the quick launch navigation for numerous project sites
all at once, you can simply create a list and "pipe" it into the
`ForEach-Object` cmdlet, as shown below:

```
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
        DeleteNavigationNode $web.Navigation.QuickLaunch "Process Guidance"

        $DebugPreference = "SilentlyContinue"
        $web = Get-SPWeb $_

        $DebugPreference = "Continue"
        ImportQuickLaunchNavigation $web $navigationXml
    }
```

The PowerShell script starts to get a little ugly due to the hack to avoid the
"Cannot complete this action" error that I mentioned earlier, but it does work.
The upgraded TFS 2005/2008 project sites will look a lot more like new project
sites created with TFS 2010.

Here is the final version of the QuickLaunch-TFS2010.xml file that I used for
upgrading my TFS project sites:

```
<QuickLaunch>
  <NavigationNode
    title="Team Web Access"
    url="($web.ServerRelativeUrl)/_layouts/tfsredirect.aspx?tf%3aType=WebAccess" />
  <NavigationNode
    title="Dashboards"
    url="($web.ServerRelativeUrl)/Dashboards/Forms/AllItems.aspx">
    <NavigationNode
      title="Burndown"
      url="($web.ServerRelativeUrl)/Dashboards/Burndown.aspx" />
    <NavigationNode
      title="Quality"
      url="($web.ServerRelativeUrl)/Dashboards/Quality.aspx" />
    <NavigationNode
      title="Bugs"
      url="($web.ServerRelativeUrl)/Dashboards/Bugs.aspx" />
    <NavigationNode
      title="Test"
      url="($web.ServerRelativeUrl)/Dashboards/Test.aspx" />
    <NavigationNode
      title="Build"
      url="($web.ServerRelativeUrl)/Dashboards/Build.aspx" />
    <NavigationNode
      title="My Dashboard"
      url="($web.ServerRelativeUrl)/Dashboards/MyDashboard.aspx" />
  </NavigationNode>
  <NavigationNode
    title="Excel Reports"
    url="($web.ServerRelativeUrl)/Reports/Forms/AllItems.aspx" />
  <NavigationNode
    title="Reports"
    url="($web.ServerRelativeUrl)/_layouts/tfsredirect.aspx?tf%3aType=ReportList" />
  <NavigationNode
    title="Libraries"
    url="($web.ServerRelativeUrl)/_layouts/viewlsts.aspx?BaseType=1">
    <NavigationNode
      title="Team Wiki"
      url="($web.ServerRelativeUrl)/Team Wiki" />
    <NavigationNode
      title="Development"
      url="($web.ServerRelativeUrl)/Development/Forms/AllItems.aspx" />
    <NavigationNode
      title="Project Management"
      url="($web.ServerRelativeUrl)/Project%20Management/Forms/AllItems.aspx" />
    <NavigationNode
      title="Requirements"
      url="($web.ServerRelativeUrl)/Requirements" />
    <NavigationNode
      title="Security"
      url="($web.ServerRelativeUrl)/Security" />
    <NavigationNode
      title="Test"
      url="($web.ServerRelativeUrl)/Test/Forms/AllItems.aspx" />
  </NavigationNode>
  <NavigationNode
    title="Lists"
    url="($web.ServerRelativeUrl)/_layouts/viewlsts.aspx?BaseType=0">
    <NavigationNode
      title="Calendar"
      url="($web.ServerRelativeUrl)/Lists/Events/calendar.aspx" />
  </NavigationNode>
  <NavigationNode
    title="Process Guidance"
    url="http://go.microsoft.com/fwlink/?LinkId=153652&amp;clcid=0x409" />
</QuickLaunch>
```
