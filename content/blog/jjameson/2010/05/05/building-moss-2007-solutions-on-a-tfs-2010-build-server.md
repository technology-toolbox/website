---
title: Building MOSS 2007 Solutions on a TFS 2010 Build Server
date: 2010-05-05T04:15:00-06:00
excerpt:
  After upgrading my Team Foundation Server (TFS) 2008 environment to TFS 2010 ,
  my next step was to upgrade various Visual Studio solutions to the 2010
  version and ensure they built successfully after the upgrade. Note that during
  the upgrade, I chose...
aliases:
  [
    "/blog/jjameson/archive/2010/05/04/building-moss-2007-solutions-on-a-tfs-2010-build-server.aspx",
    "/blog/jjameson/archive/2010/05/05/building-moss-2007-solutions-on-a-tfs-2010-build-server.aspx",
  ]
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "Visual Studio", "TFS"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/05/05/building-moss-2007-solutions-on-a-tfs-2010-build-server.aspx"
---

After
[upgrading my Team Foundation Server (TFS) 2008 environment to TFS 2010](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010-overview),
my next step was to upgrade various Visual Studio solutions to the 2010 version
and ensure they built successfully after the upgrade.

Note that during the upgrade, I chose to rebuild the VM (DAZZLER) in my
environment that runs the Team Foundation Build Service. In other words, after
backing up the VM (and robocopying the Builds folder to a temporary location on
a different server), I shutdown DAZZLER and overwrote the VHD file with my
SysPrep'ed image of Windows Server 2008 R2.

Upon starting the VM again -- and completing the "mini-setup" -- I joined the
machine to the domain and proceeded to install the 2010 version of Team
Foundation Build Service (as detailed in
[one of yesterday's posts](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010)).
Then I installed Visual Studio 2010 on the build server.

However, after the rebuild, I forgot to configure the environment to build a
solution for Microsoft Office SharePoint Server (MOSS) 2007. (A long time ago, I
had copied the SharePoint assemblies to DAZZLER in order to compile MOSS 2007
solutions.)

Consequently, when I queued up a build for my Fabrikam.Demo solution, I
encountered errors because the referenced assemblies (e.g.
Microsoft.SharePoint.dll) were no longer available on the build server.

To configure the build server to compile MOSS 2007 solutions:

1. Create a folder on the build server to contain the referenced assemblies for
   MOSS 2007:\
   \
   **C:\Program Files\Reference Assemblies\Microsoft\SharePoint v3**
1. Copy the referenced SharePoint assemblies from another VM (which has MOSS
   2007 installed) to this new folder. The list of SharePoint assemblies that
   you need to copy depends on the details of your solution, but typically
   includes:

   - Microsoft.SharePoint.dll
   - Microsoft.SharePoint.Publishing.dll
   - Microsoft.SharePoint.Security.dll
   - ...
1. Create a corresponding registry key for MSBuild to locate the reference
   assemblies. This is most easily accomplished by running the following from an
   Administrator command prompt:

   {{< console-block >}}

   reg add
   "HKLM\SOFTWARE\Wow6432Node\Microsoft\\.NETFramework\v2.0.50727\AssemblyFoldersEx\SharePoint
   v3" /d "C:\Program Files\Reference Assemblies\Microsoft\SharePoint v3"

   {{< /console-block >}}

After completing these steps, I queued another build of my Fabrikam.Demo
solution and verified the errors caused by missing SharePoint assemblies no
longer occurred.

{{< div-block "note" >}}

> **Note**
>
> In addition to copying SharePoint assemblies that are directly referenced in
> your projects, you also may want to copy assemblies that are _indirectly_
> referenced, such as:
>
> - Microsoft.HtmlTrans.Interface.dll
> - Microsoft.Internal.Mime.dll
> - Microsoft.Office.Server.dll
> - Microsoft.SharePoint.AdministrationOperation.dll
> - Microsoft.SharePoint.Diagnostics.dll
> - Microsoft.SharePoint.Dsp.dll
> - Microsoft.SharePoint.Library.dll
> - Microsoft.SharePoint.Search.dll
> - Microsoft.Web.Design.Server.dll
>
> While not required to successfully build a SharePoint solution, copying these
> additional assemblies will avoid warnings during the build.
>
> If you choose to include these additional assemblies, be aware that many of
> these files will need to be copied out of the GAC on the MOSS 2007 server (in
> other words, most of them are not located in the "Program Files\Common
> Files\microsoft shared\Web Server Extensions\12\ISAPI" folder).

{{< /div-block >}}
