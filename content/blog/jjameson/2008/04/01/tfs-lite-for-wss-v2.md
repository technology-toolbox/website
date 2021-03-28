---
title: "\"TFS Lite\" for WSS v2"
date: 2008-04-01T07:02:00-06:00
excerpt:
  "[Update (2008-04-07): \"TFS Lite\" for WSS v3 is now available.] For almost
  as long as I can remember (okay, not really that long -- but at least as far
  back as 2003), I've been using SharePoint lists as a bug tracking tool on
  almost all of the customer..."
aliases:
  [
    "/blog/jjameson/archive/2008/03/31/tfs-lite-for-wss-v2.aspx",
    "/blog/jjameson/archive/2008/04/01/tfs-lite-for-wss-v2.aspx",
  ]
categories: ["Development", "SharePoint"]
tags: ["Core Development", "WSS v2"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2008/04/01/tfs-lite-for-wss-v2.aspx"
attachment: 
  url: "https://assets.technologytoolbox.com/blog/jjameson/Documents/TFS Lite - WSS v2.zip"
  fileName: TFS Lite - WSS v2.zip
  fileSizeInBytes: 230157
---

{{< div-block "note update" >}}

> **Update (2008-04-07)**
>
> ["TFS Lite" for WSS v3](/blog/jjameson/2008/04/07/tfs-lite-for-wss-v3) is now
> available.

{{< /div-block >}}

For almost as long as I can remember (okay, not really that long -- but at least
as far back as 2003), I've been using SharePoint lists as a bug tracking tool on
almost all of the customer projects that I have been involved with. Contrary to
what you might think, many of our enterprise customers do not have effective
tools and processes in place for managing a software development project.

Long before I ever even heard of "Whitehorse" and "Burton" -- now known by their
official monikers, Visual Studio Team System (VSTS) and Team Foundation Server
(TFS) -- I discovered that Windows SharePoint Services v2 included the essential
core features for tracking bugs and change requests. I remember sitting down and
customizing the out-of-the-box **Issues** list in WSS v2 so that the fields were
essentially identical those in Product Studio, which -- at that time -- was the
primary tool used by the product groups in Redmond (Product Studio was the
successor to RAID). I added some new fields (for example, **Area**, **Triage**,
and **Severity**) and modified existing fields, such as changing the
**Priority** choices to **1 - High**, **2 - Medium**, and **3 - Low** (to sort
by the preferred order by default).

The beauty of doing this in WSS v2 is that a) it is incredibly easy to setup,
and b) it is a supported product that requires no custom code whatsoever. In
other words, there is no need to download and learn one of the many freely
available (but not commercially supported) bug tracking tools. In fact, the
ability to save a site (or just a single list) as a template meant that I could
configure a new bug tracking "application" for a customer within a matter of
minutes.

Fast forward a couple of years...

When VSTS and TFS were launched, I modified my site template to closely align
with the Work Item Tracking features, even going so far as to dub the site
template "TFS Lite." At this point, you might be asking, "Jeremy, why didn't you
simply recommend that your customers use TFS instead of this 'home baked'
solution built on a WSS list?"

What you have to remember, however, is that Microsoft was somewhat slow to
realize that TFS was a great fit for both small workgroups and large enterprise
organizations as well. In fact, I do not believe the TFS Workgroup Edition was
even available until about a year after TFS originally shipped. It is also
important to note -- at least from my experience -- that the effort in getting
customers to commit to using TFS is an order of magnitude greater than getting
them to use SharePoint alone. In other words, it simply isn't (or at least
wasn't at the time) always feasibile to use TFS.

Please don't misunderstand what I am saying. TFS is an incredibly powerful tool,
and my simple "TFS Lite" solution is nowhere near as feature rich as the full
product. If there is any possibility of using TFS in your organization or on
your project, then you should most certainly push for that option instead
(especially now that the entry-level TFS Workgroup Edition is available for 5
users or less).

Okay, enough with the history and disclaimers. Let's get to the interesting
stuff. The following overview of the TFS Lite template is pulled from a document
I wrote back in August 2006. I have also attached the SharePoint site template
for WSS v2.

### "TFS Lite" Site Template for SharePoint

The primary feature of the "TFS Lite" template is the **Work Items** list, which
is loosely based on the work item tracking features in Visual Studio Team
Foundation Server (hence the name of the site template).

The primary goal in developing the TFS Lite template was to support a "project
dashboard" similar to Figure 1 _with minimal effort by the various team
members_. In other words, as work items in the underlying SharePoint list are
updated, the project summary should automatically reflect the changes.

Another key feature is the ability to show the status of important project
deliverables and milestones using KPI icons (green, yellow, and red).

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/TFS-Lite-WSS-v2-600x452.jpg"
alt="Project Summary \"dashboard\"" class="screenshot" height="452" width="600"
title="Figure 1: Project Summary \"dashboard\"" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/TFS-Lite-WSS-v2-788x594.jpg)

#### Work Items List

The heart of the TFS Lite template is the custom **Work Items** list, which is
based on the default SharePoint **Issues** list (in order to maintain the
history as items are modified). The columns in the list have been customized to
closely match the **MSF for Agile Software Development** template in Team
Foundation Server (TFS).

For example, the **Area** field has been added to associate a work item with
different areas of the solution. Similarly, the **Iteration** field has been
added to group work items into different phases of the project.

{{< div-block "note" >}}

> **Note**
>
> The **Area** and **Iteration** fields should be customized for each project to
> reflect the major work effort areas (e.g. features) and milestones.

{{< /div-block >}}

In addition to modifying the columns in the default Issues list, the Work Items
list also supports several additional views (similar to the queries provided in
TFS).

#### Project Summary

The overall project status displayed on the dashboard is simply a Content Editor
Web Part with a small amount of HTML. This option was selected based on the ease
of the occasional need to modify a little HTML compared with the complexity of
"calculating" the project status (or storing it in some other list).

The **Project Summary** Web Part on the dashboard began as a simple List View
Web Part using the following criteria:

> **Current** is equal to **Yes**\
> And **Exit Criteria** is equal to **Yes**\
> And **Iteration** is equal to **Project\v1.0\M0**

Note that the **Exit Criteria** field is used to denote work items as
significant deliverables or milestones (thereby excluding less important work
items from the **Project Summary**). Also note that **Iteration** is used to
filter important items that are not scheduled to be worked on until a later
phase.

The **Project Summary** Web Part on the dashboard was developed with a "minimal
code approach." Rather than modifying the underlying CAML for the Work Items
list (an alternate approach to display an image corresponding to the KPI status)
FrontPage was used to convert the List View Web Part to a Data View Web Part
(DVWP). The DVWP was then exported to a file and then imported it to the
dashboard page (thus avoiding "unghosting" the entire Project Summary page).

Note that the approach of using a DVWP instead of modifying the underlying CAML
introduces a hard-coded list ID (a GUID). Consequently the list data source will
need to be tweaked after create a new site based on this template.

A minimal amount of custom XSLT was then added to display an image based on the
KPI value of each work item:

```XSLT
<xsl:choose>
  <xsl:when test="@KPI='Green'">
    <img alt="Green" src="/sites/Frontier/Image%20Library/kpinormal-0.gif" />
  </xsl:when>
  <xsl:when test="@KPI='Yellow'">
    <img alt="Yellow" src="/sites/Frontier/Image%20Library/kpinormal-1.gif" />
  </xsl:when>
  <xsl:when test="@KPI='Red'">
    <img alt="Red" src="/sites/Frontier/Image%20Library/kpinormal-2.gif" />
  </xsl:when>
  <xsl:otherwise>
    <xsl:value-of disable-output-escaping="no" select="@KPI" />
  </xsl:otherwise>
</xsl:choose>
```

Note that the image paths are hard-coded to use a picture library (named "Image
Library") in the same site. An alternative would be to store these images in the
\_layout folder, however that approach would require an Administrator to deploy
the images.

#### Accomplishments

The **Accomplishments** Web Part on the dashboard is a simple List View Web Part
using the following criteria:

> **Current** is equal to **Yes**\
> And **Status** is equal to **Closed**\
> And **ModifiedFilter** is greater than or equal to **[Today]**

Note that **ModifiedFilter** is a calculated column simply used to filter out
items closed more than a week ago.

#### Top 10 Issues

The **Top 10 Issues** Web Part on the dashboard is a simple List View Web Part
using the following criteria:

> **Current** is equal to **Yes**\
> And **Blocked** is equal to **Yes**

Note that the **Blocked** field indicates there is an issue in completing the
work item.

#### Priorities/Milestones

The criteria for **Priorities/Milestones** Web Part is similar to the **Project
Summary** Web Part:

> **Current** is equal to **Yes**\
> And **Exit Criteria** is equal to **Yes**\
> And **Status** is not equal to **Closed**

Also note that grouping is used to loosely sort the major deliverables and
milestones.
