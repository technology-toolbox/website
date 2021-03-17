---
title: "Configuring Quick Launch Navigation in SharePoint Server 2010 Using PowerShell"
date: 2010-05-17T05:43:00-06:00
excerpt:
  "Suppose that you need to update a few SharePoint team sites to add a couple
  of links to the quick launch navigation. Assuming the number of sites to be
  updated is relatively small, then it is reasonable to manually apply the
  configuration changes via..."
aliases: ["/blog/jjameson/archive/2010/05/16/configuring-quick-launch-navigation-in-sharepoint-server-2010-using-powershell.aspx", "/blog/jjameson/archive/2010/05/17/configuring-quick-launch-navigation-in-sharepoint-server-2010-using-powershell.aspx"]
draft: true
categories: ["SharePoint"]
tags: ["SharePoint 2010", "PowerShell"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/05/17/configuring-quick-launch-navigation-in-sharepoint-server-2010-using-powershell.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/05/17/configuring-quick-launch-navigation-in-sharepoint-server-2010-using-powershell.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Suppose that you need to update a few SharePoint team sites to add a couple of
links to the quick launch navigation. Assuming the number of sites to be updated
is relatively small, then it is reasonable to manually apply the configuration
changes via the **Site Settings** page on each site.

However, what if you need to update a few dozen sites -- or, even worse, more
than a hundred sites? You can imagine this "little change" would quickly seem
daunting (or, at the very least, completely mind numbing to carry out manually).

In the past -- i.e. when working with Microsoft Office SharePoint Server (MOSS)
2007 -- I would have cranked out a little C# code to perform mundane tasks like
this. That is, of course, assuming the amount of time necessary to write the
code would be roughly equivalent to the time required for me to complete the
steps manually.

Now, with SharePoint Server 2010, we have an entirely new option for automating
configuration changes like this -- namely PowerShell. [True, if you want to do
some of the "heavy lifting" yourself, you can also use PowerShell with MOSS
2007.]

I've recently written a
[series of posts about upgrading from Team Foundation Server (TFS) 2008 to to TFS 2010](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010-overview)
-- including some details about
[upgrading the project sites hosted in SharePoint](/blog/jjameson/2010/05/14/upgrading-tfs-2005-2008-project-sites-to-tfs-2010-part-1-agile-dashboard-features).
During this process, I decided that I'd update my "legacy" TFS project sites to
have the same quick launch navigation as new project sites created in TFS 2010
with the MSF Agile v5 process template.

A few years ago, I wrote some code to manipulate the navigation elements on a
team site when a custom feature was activated. Consequently, I was already
familiar with the portion of the SharePoint object model that I needed to use in
order to modify the quick launch navigation.

However, I'm still very new to the whole PowerShell thing, and I viewed this as
a great "real world" opportunity to improve my skills. [As such, if you see
areas for improvement in my PowerShell scripts, please don't hesitate to add a
comment and let me know.]

I started by writing some PowerShell script to export the quick launch
navigation elements for a specific site to XML:

```
# Exports the quick launch navigation for a SharePoint site as XML

function ExportQuickLaunchNavigation(
    [Microsoft.SharePoint.SPWeb] $web)
{
    $xml = [xml] "<QuickLaunch/>"

    foreach ($navigationNode in $web.Navigation.QuickLaunch)
    {
        AddNavigationElement $navigationNode $xml.DocumentElement
    }

    return $xml
}

function AddNavigationElement(
    [Microsoft.SharePoint.Navigation.SPNavigationNode] $navigationNode,
    [System.Xml.XmlElement] $parentElement)
{
    $navElement = $parentElement.OwnerDocument.CreateElement("NavigationNode")

    $parentElement.AppendChild($navElement) > $null

    $navElement.SetAttribute("title", $navigationNode.Title)
    $navElement.SetAttribute("url", $navigationNode.Url)

    foreach ($childNode in $navigationNode.Children)
    {
        AddNavigationElement $childNode $navElement
    }
}

$web = Get-SPWeb "http://foobar3/sites/Test"

$navigationXml = ExportQuickLaunchNavigation($web)

$navigationXml.OuterXml
```

Assuming the referenced site is based on the out-of-the-box Team Site template,
the above PowerShell will output the following XML (without the nice formatting,
of course):

```
<QuickLaunch>
  <NavigationNode
      title="Libraries"
      url="/sites/Test/_layouts/viewlsts.aspx?BaseType=1">
    <NavigationNode title="Site Pages" url="/sites/Test/SitePages" />
    <NavigationNode
      title="Shared Documents"
      url="/sites/Test/Shared Documents/Forms/AllItems.aspx" />
  </NavigationNode>
  <NavigationNode
      title="Lists"
      url="/sites/Test/_layouts/viewlsts.aspx?BaseType=0">
    <NavigationNode
      title="Calendar"
      url="/sites/Test/Lists/Calendar/calendar.aspx" />
    <NavigationNode title="Tasks" url="/sites/Test/Lists/Tasks/AllItems.aspx" />
  </NavigationNode>
  <NavigationNode
      title="Discussions"
      url="/sites/Test/_layouts/viewlsts.aspx?BaseType=0&amp;ListTemplate=108">
    <NavigationNode
      title="Team Discussion"
      url="/sites/Test/Lists/Team Discussion/AllItems.aspx" />
  </NavigationNode>
</QuickLaunch>
```

Note that each `<NavigationNode>` element may contain child `<NavigationNode>`
elements (representing the hierarchical nature of the quick launch navigation).

The above XML represents the following quick launch navigation:

- Libraries
  - Site Pages
  - Shared Documents
- Lists
  - Calendar
  - Tasks
- Discussions
  - Team Discussion

Once I had the export working, I then proceeded to work on the import process.

The "requirements" that I established for the import process are as follows:

- Using the navigation link URL as the key (i.e. the `url` attribute of each
  `<NavigationNode>` element), ensure each navigation node specified in the XML
  exists in the quick launch navigation.
- Ensure the order of the links in the quick launch navigation matches the order
  specified in the XML.
- Ignore any links in the quick launch navigation on the site that are not
  specified in the XML (in other words, don't delete links if there are no
  corresponding elements in the XML).

The last item ensures that any custom links that might have been added to a team
site are preserved.

To understand how the import process should work, consider the following input
XML:

```
<QuickLaunch>
  <NavigationNode
    title="My MSDN Blog"
    url="http://blogs.msdn.com/jjameson">
    <NavigationNode
      title="My MSDN Blog - Dashboard"
      url="http://blogs.msdn.com/controlpanel/blogs/default.aspx" />
  </NavigationNode>
  <NavigationNode
    title="Team Web Access"
    url="/sites/AdventureWorks/_layouts/tfsredirect.aspx?tf%3aType=WebAccess" />
</QuickLaunch>
```

Note that in this example, all of the navigation nodes are considered to be
"new" (since none of them match the ones specified in the earlier output).
Consequently, we should expect the quick launch navigation to resemble the
following after the import completes:

- My MSDN Blog
  - My MSDN Blog - Dashboard
- Team Web Access
- Libraries
  - Site Pages
  - Shared Documents
- Lists
  - Calendar
  - Tasks
- Discussions
  - Team Discussion

Here is the corresponding PowerShell script to import the quick launch
navigation for a site:

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
    [System.Xml.XmlNodeList] $navElements)
{
    [int] $position = 0

    foreach ($navElement in $navElements)
    {
        $title = $navElement.GetAttribute("title")
        $url = $navElement.GetAttribute("url")

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
            ImportNavigationNodes $navigationNode.Children $childNodes
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
        ImportNavigationNodes $nodes $navElements
    }
    Else
    {
        Write-Host "No navigation nodes found to import."
    }
}

$navigationXml = [xml] @"
<QuickLaunch>
  <NavigationNode
    title="My MSDN Blog"
    url="http://blogs.msdn.com/jjameson">
    <NavigationNode
      title="My MSDN Blog - Dashboard"
      url="http://blogs.msdn.com/controlpanel/blogs/default.aspx" />
  </NavigationNode>
  <NavigationNode
    title="Team Web Access"
    url="/sites/AdventureWorks/_layouts/tfsredirect.aspx?tf%3aType=WebAccess" />
</QuickLaunch>
"@

$DebugPreference = "SilentlyContinue"
$web = Get-SPWeb "http://foobar3/sites/Test"

$DebugPreference = "Continue"
ImportQuickLaunchNavigation $web $navigationXml
```

If I wanted to add the new links after any existing navigation links, I would
either need to specify the existing links in the input XML (which wouldn't
require any changes to the corresponding PowerShell script) or extend the XML
schema to support some sort of "position" attribute (which would add significant
complexity to the script).

In my
[next post](/blog/jjameson/2010/05/17/upgrading-tfs-2005-2008-project-sites-to-tfs-2010-part-3-quick-launch-navigation),
I'll show how you can use PowerShell to update the quick launch navigation for
TFS 2005/2008 project sites upgraded to TFS 2010.
