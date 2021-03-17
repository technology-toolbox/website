---
title: "Create a Custom \"Builds\" List on Your TFS Project Portal (a.k.a. SharePoint Team Site)"
date: 2010-11-29T06:11:00-07:00
excerpt: "One of \"tweaks\" that I commonly make to the SharePoint team site created for each project in Team Foundation Server is to create a custom list to track the important builds for the project (typically corresponding to each milestone or iteration). 
 There..."
aliases: ["/blog/jjameson/archive/2010/11/28/create-a-custom-quot-builds-quot-list-on-your-tfs-project-portal-a-k-a-sharepoint-team-site.aspx", "/blog/jjameson/archive/2010/11/29/create-a-custom-quot-builds-quot-list-on-your-tfs-project-portal-a-k-a-sharepoint-team-site.aspx"]
draft: true
categories: ["My System", "Development"]
tags: ["My System", "TFS"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/11/29/create-a-custom-quot-builds-quot-list-on-your-tfs-project-portal-a-k-a-sharepoint-team-site.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/11/29/create-a-custom-quot-builds-quot-list-on-your-tfs-project-portal-a-k-a-sharepoint-team-site.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

One of "tweaks" that I commonly make to the SharePoint team site created for
each project in Team Foundation Server is to create a custom list to track the
important builds for the project (typically corresponding to each milestone or
iteration).

There's really nothing special about the list. It's just a custom list (named
**Builds**) that appears in the Quick Launch navigation and has two columns on
the default view:

- **Title** (Single line of text, default column)
- **Version** (Single line of text)

I use the default Title field to specify the milestone or iteration (e.g.
Sprint-1, Sprint-2, etc.) and the custom Version field to specify the assembly
version of the build (technically the
[Assembly File Version](/blog/jjameson/2009/04/03/best-practices-for-net-assembly-versioning)).

Shortly before or after a release to the Production environment (i.e. whenever
the build number of the release is known), I add a new item to the list. If the
build number changes (for example, because of a QFE/hotfix), I update the
version on the existing list item. [Note that if you want to view the history of
a particular item in the list to see who changed what and when the change was
made, you can enable Item Version History on the list. Simply click **Settings**
--&gt; **List Settings**, and then in the **General Settings** section, click
**Versioning settings**.]

For example, here are the contents of the list for my current project:

{{< table class="small" caption="Builds" >}}

| Title | Version |
| --- | --- |
| Sprint-1 | 1.0.7.0 |
| Sprint-2 | 1.0.41.0 |
| Sprint-3 | 1.0.51.3 |
| Sprint-4 | 1.0.81.0 |
| Sprint-5 | 1.0.116.0 |
| Sprint-6 | 1.0.168.0 |
| Sprint-7 | 1.0.237.0 |
| Sprint-8 | 1.0.270.0 |
| Sprint-9 | 1.0.295.5 |

{{< /table >}}

This list makes it very easy for anyone with access to the team site to
determine which build was associated with a particular release.

I suppose that I could spend a few seconds renaming the Title column to
something like Milestone or Iteration, but honestly I don't bother (at least not
anymore).
