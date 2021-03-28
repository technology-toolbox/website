---
title: "\"Build Bloat\" and Removing Extraneous Items from TFS Builds"
date: 2010-01-12T08:26:00-07:00
excerpt:
  This week I am wrapping up the third sprint (a.k.a. iteration or milestone )
  on my current Microsoft Office SharePoint Server (MOSS) 2007 project.
  Although, honestly, I wasn't involved all that much in Sprint-3, since I was
  on vacation for the vast majority...
aliases:
  [
    "/blog/jjameson/archive/2010/01/11/build-bloat-and-removing-extraneous-items-from-tfs-builds.aspx",
    "/blog/jjameson/archive/2010/01/12/build-bloat-and-removing-extraneous-items-from-tfs-builds.aspx",
  ]
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "WSS v3", "TFS"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/01/12/build-bloat-and-removing-extraneous-items-from-tfs-builds.aspx"
---

This week I am wrapping up the third sprint (a.k.a. _iteration_ or _milestone_)
on my current Microsoft Office SharePoint Server (MOSS) 2007 project. Although,
honestly, I wasn't involved all that much in Sprint-3, since I was on vacation
for the vast majority of the iteration.

One of the surprises that I discovered upon my return was that the size of our
builds had increased substantially since the previous iteration.

Here's a summary of the various builds for the iterations so far:

- Sprint-1: 13.4 MB uncompressed (3.2 MB compressed)
- Sprint-2: 79 MB uncompressed (18 MB compressed)
- Sprint-3: 127 MB uncompressed (48 MB compressed)

As you can see, while the size of the Sprint-1 build was rather paltry, the
Sprint-3 build was almost 10 times larger (uncompressed). The compressed version
is nearly 16 times larger.

Not suprisingly, I refer to this issue as "build bloat."

This is painfully apparent when copying a build across the Internet (for
example, across the globe to our Test environment in a datacenter in Singapore,
or even merely across the country to our Production environment).

It really isn't surprising that the size of the builds for subsequent sprints
are signficantly larger than the Sprint-1 build. After all, in Sprint-2, I
introduced a "modal popup framework" based on the
[Microsoft AJAX Control Toolkit](http://www.asp.net/ajax) in order to show
announcements to people browsing the site, and the AjaxControlToolkit.dll
assembly is approximately 4 MB.

Similarly, for Sprint-3 we started using additional third-party controls for
handling some of the more sophisticated user interface elements, and the
Telerik.Web.UI.dll assembly is currently a whopping 14 MB.

At this point, you might be wondering how the addition of a 4 MB assembly and a
14 MB assembly increased the size of the build approximately 113 MB.

As I noted at the beginning of this post, this is a MOSS 2007 project.
Consequently, we deploy our solution using Web Solution Packages (WSPs). Note
that a WSP is really just a CAB file with a different file extension.
Consequently, when we build our WSPs, we include not only our custom SharePoint
features in the CAB file, but also any dependencies (e.g. referenced assemblies
like AjaxControlToolkit.dll).

One solution to the problem of build bloat would be to simply remove the
referenced assemblies and deploy those separately. However, I really don't like
the idea of breaking the nice encapsulation of deploying a WSP and having the
corresponding referenced assemblies automatically deployed as well. Sure, one
could argue that deploying third-party controls like the AJAX Control Toolkit
and the Telerik controls should be done similar to the .NET Framework itself
(i.e. independent of our custom code), but my preference is to simplify the
deployment -- from a Release Management perspective -- as much as possible.

In addition, it is also important to note that the build bloat isn't caused
entirely by the UI control assemblies. As I've
[noted in the past](/blog/jjameson/2009/03/30/extraneous-sharepoint-assemblies),
SharePoint has a bad habit of copying extraneous assemblies into your project.

In this particular case, Microsoft.Office.Server.Search.dll and
Microsoft.SharePoint.Server.Search.dll are now included in each build -- even
though we really don't want them to be (since we would never install or update
these files as part of our deployment process for our solution).

Note that the build bloat is exacerbated by compiling both Debug and Release
builds, as well as the fact that the default build configuration for Team
Foundation Server (TFS) copies the files for any Web site projects to a separate
folder (_PublishedWebsites).

When I started investigating the build bloat issue, I quickly discovered that
there were actually five copies each of AjaxControlToolkit.dll and
Telerik.Web.UI.dll.

To resolve the build bloat issue, I modified our TFSBuild.proj file to override
the `BeforeDropBuild` target:

```XML
<Target Name="BeforeDropBuild">
  <Message Importance="high"
    Text="Removing extraneous files from the build..." />

  <!-- We do not currently deploy anything from the _PublishedWebsites folders -->
  <RemoveDir
    Directories="$(BinariesRoot)\Debug\_PublishedWebsites;
      $(BinariesRoot)\Release\_PublishedWebsites;" />

  <!--
    The following assemblies are either included in the WSPs
    or else should not be included in the build
    (e.g. Microsoft.Office.Server.Search.dll).
  -->
  <CreateItem
    Include="$(BinariesRoot)\**\AjaxControlToolkit.dll;
      $(BinariesRoot)\**\Microsoft.Office.Server.Search.dll;
      $(BinariesRoot)\**\Microsoft.SharePoint.Server.Search.dll;
      $(BinariesRoot)\**\Telerik.Web.UI.dll" >
    <Output TaskParameter="Include" ItemName="FilesToDelete"/>
  </CreateItem>

  <Delete Files="@(FilesToDelete)" />
</Target>
```

Note that by default, any Web projects in your solution are automatically copied
to the \_PublishedWebsites folder (so that you can do an XCOPY deployment of the
Web sites, if you wish). However, since this solution is based on SharePoint, we
obviously are not deploying the Web site in that way. Consequently, I chose to
remove those folders completely.

Next, I recursively delete any extraneous assemblies (either because they are
included in a WSP or because we would never install them from the build).

After overriding the `BeforeDropBuild` target, the Sprint-3 build is now 28 MB
uncompressed (17 MB compressed) -- compared to 127 MB uncompressed (48 MB
compressed) before.

Buh-bye, build bloat!

{{< div-block "note update" >}}

> **Update (2011-03-14)**
>
> Refer to
> [one of my later posts](/blog/jjameson/2011/03/14/quot-build-bloat-quot-part-2-a-k-a-removing-extraneous-items-from-sharepoint-visual-studio-projects)
> for an update on this technique that works with TFS 2010 builds that don't use
> the UpgradeTemplate.xaml build process template.

{{< /div-block >}}
