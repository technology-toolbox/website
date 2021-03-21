---
title: "\"Build Bloat\", Part 2 (a.k.a. Removing Extraneous Items from SharePoint Visual Studio Projects)"
date: 2011-03-14T06:54:00-06:00
excerpt:
  "Last week I received a \"Logical Disk Free Space is low\" alert from
  Operations Manager for my TFS 2010 build server (DAZZLER). After a few minutes
  investigating the issue, I discovered that my \"Builds\" folder was consuming
  a little over 2 GB of storage..."
aliases:
  [
    "/blog/jjameson/archive/2011/03/13/quot-build-bloat-quot-part-2-a-k-a-removing-extraneous-items-from-sharepoint-visual-studio-projects.aspx",
    "/blog/jjameson/archive/2011/03/14/quot-build-bloat-quot-part-2-a-k-a-removing-extraneous-items-from-sharepoint-visual-studio-projects.aspx",
  ]
draft: true
categories: ["My System", "SharePoint", "Development"]
tags: ["My System", "MOSS 2007", "Visual Studio", "TFS", "SharePoint 2010"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2011/03/14/quot-build-bloat-quot-part-2-a-k-a-removing-extraneous-items-from-sharepoint-visual-studio-projects.aspx"
---

Last week I received a <q class="directQuote">"Logical Disk Free Space is
low"</q> alert from Operations Manager for my TFS 2010 build server (DAZZLER).

After a few minutes investigating the issue, I discovered that my "Builds"
folder was consuming a little over 2 GB of storage. Note that DAZZLER only has a
23 GB virtual hard drive allocated to it -- which, by the time you install the
Team Foundation Server Build service and Visual Studio 2010 -- doesn't have a
great deal of extra capacity to begin with.

However, there's certainly no reason that I should need anywhere near 2 GB
simply for storing the build output from my various sample solutions (like
Fabrikam Demo and Tugboat).

I quickly identified the crux of the issue by looking at a couple of the builds
for the Tugboat solution. I noticed that a recent build of the Tugboat solution
was almost 76 MB on disk. If you've seen the
[Tugboat sample](/blog/jjameson/tags/Tugboat/), then you should know it
certainly doesn't justify this amount of disk space.

It turns out that 73 MB in each build folder was consumed by multiple copies of
SharePoint assemblies, such as:

- Microsoft.SharePoint.AdministrationOperation.dll
- Microsoft.SharePoint.Library.dll
- Microsoft.Internal.Mime.dll
- etc.

In other words, it's essentially the same
[problem I blogged about over a year ago](/blog/jjameson/2010/01/12/build-bloat-and-removing-extraneous-items-from-tfs-builds)
-- with a slight twist.

In my earlier post, I described adding a `BeforeDropBuild` target to the
TFSBuild.proj file in order to remove extraneous assemblies from the build.
However, since I'm now using TFS 2010 build (and not using the
UpgradeTemplate.xaml build process template), the TFSBuild.proj is no longer
used. Hence a `BeforeDropBuild` target no longer does the trick.

Fortunately, the solution is very easy. I simply copied the target that I
created previously and pasted it into the individual Visual Studio project files
(renaming the target in the process). Instead of hooking into `BeforeDropBuild`
(which is meaningless when compiling Visual Studio projects), I now hook into
the `AfterBuild` target.

I think this makes more sense with an example.

Here's the corresponding pieces from my updated Web.csproj file for the Tugboat
SharePoint sample:

```XML
  <Target Name="AfterBuild">
    <CallTarget Targets="RemoveExtraneousFilesFromBuild" />
  </Target>
  <Target Name="RemoveExtraneousFilesFromBuild">
    <Message Importance="high" Text="Removing extraneous files from build output ($(OutDir))..." />
    <!--
      The following assemblies are either included in the WSPs
      or else should not be included in the build
      (e.g. Microsoft.SharePoint.dll).
    -->
    <CreateItem Include="$(OutDir)\Microsoft.BusinessData.dll;
      $(OutDir)\Microsoft.HtmlTrans.Interface.dll;
      $(OutDir)\Microsoft.IdentityModel.dll;
      $(OutDir)\Microsoft.Internal.Mime.dll;
      $(OutDir)\Microsoft.Office.Server.dll;
      $(OutDir)\Microsoft.Office.Server.UI.dll;
      $(OutDir)\Microsoft.Office.Server.UserProfiles.dll;
      $(OutDir)\Microsoft.SharePoint.AdministrationOperation.dll;
      $(OutDir)\Microsoft.SharePoint.Client.ServerRuntime.dll;
      $(OutDir)\Microsoft.SharePoint.Diagnostics.dll;
      $(OutDir)\Microsoft.SharePoint.dll;
      $(OutDir)\Microsoft.SharePoint.Dsp.dll;
      $(OutDir)\Microsoft.SharePoint.Library.dll;
      $(OutDir)\Microsoft.SharePoint.PowerShell.dll;
      $(OutDir)\Microsoft.SharePoint.Publishing.dll;
      $(OutDir)\Microsoft.SharePoint.Search.dll;
      $(OutDir)\Microsoft.SharePoint.Security.dll;
      $(OutDir)\Microsoft.SharePoint.Taxonomy.dll;
      $(OutDir)\Microsoft.Web.Administration.dll;
      $(OutDir)\Microsoft.Web.CommandUI.dll;
      $(OutDir)\Microsoft.Web.Design.Server.dll;">
      <Output TaskParameter="Include" ItemName="FilesToDelete" />
    </CreateItem>
    <Delete Files="@(FilesToDelete)" />
  </Target>
```

Note that I'm now using `$(OutDir)`-- instead of `$(BinariesRoot)` -- in order
to remove the assemblies from the build output folders.

With this change, my Tugboat builds are now a mere 1.13 MB on disk. Woohoo!

After purging the bloated builds for the Tugboat and Fabrikam Demo solutions,
I've managed to reclaim a couple of gigabytes of precious VHD space on my TFS
build server.
