---
title: Building SharePoint WSPs with Team Foundation Build
date: 2009-11-18T05:45:00-07:00
excerpt:
  As I noted in my previous post , I recently discovered that my approach for
  building Web Solution Packages (WSPs) in Microsoft Office SharePoint Server
  (MOSS) 2007 isn't compatible with Team Foundation Build. I'm actually a little
  embarrassed to say...
aliases:
  [
    "/blog/jjameson/archive/2009/11/17/building-sharepoint-wsps-with-team-foundation-build.aspx",
    "/blog/jjameson/archive/2009/11/18/building-sharepoint-wsps-with-team-foundation-build.aspx",
  ]
draft: true
categories: ["My System", "SharePoint", "Development"]
tags: ["My System", "Simplify", "MOSS 2007", "WSS v3", "TFS"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/11/18/building-sharepoint-wsps-with-team-foundation-build.aspx"
---

As I noted in my
[previous post](/blog/jjameson/2009/11/18/the-copy-local-bug-in-visual-studio),
I recently discovered that
[my approach for building Web Solution Packages (WSPs)](/blog/jjameson/2009/09/28/sample-walkthrough-of-the-dr-dada-approach-to-sharepoint)
in Microsoft Office SharePoint Server (MOSS) 2007 isn't compatible with Team
Foundation Build.

I'm actually a little embarrassed to say this, but when I created the original
["DR.DADA" approach for MOSS 2007](/blog/jjameson/2009/03/31/introducing-the-dr-dada-approach-to-sharepoint-development)
development back on the Agilent Technologies project, we were using Visual
SourceSafe -- not Team Foundation Server (TFS) -- and a "manual" build process.

I'd used VSS and automated builds on other projects before (using NAnt), but
never got around to automating our MOSS 2007 builds on the Agilent project
because, honestly, there were just too many other higher priority items.
Besides, each build only required a couple of minutes of actual human effort
because most of the build was scripted.

Still, an automated daily build (and deployment to DEV) is a
[really, really good thing to have](/blog/jjameson/2009/09/26/best-practices-for-scm-and-the-daily-build-process).

I've been fortunate to be on a few projects since then that have leveraged TFS.

However, up until about a month ago, I hadn't used Team Foundation Build
(outside of the
[Jameson Datacenter](/blog/jjameson/2009/09/14/the-jameson-datacenter), of
course) due to the fact that we are leveraging the extranet TFS instance hosted
by Microsoft.

Note that Microsoft IT makes it very easy for us to provision new TFS projects
on either the extranet or one of several internal TFS instances. Configuring
builds using Team Foundation Build on one of the intranet TFS instances is very
easy (from what I hear), but I strongly prefer working off the extranet TFS
instance because then I don't have to VPN into CorpNet in order to have access
to source control.

However, choosing the extranet TFS instance also means we can't configure builds
using the out-of-the-box functionality in Team Foundation Server (at least not
without setting up a build server on the extranet). Fortunately, I've found a
way to schedule "manual" builds that look a lot like automated builds performed
using Team Foundation Build.

So, if you are building SharePoint WSPs -- regardless of whether you use the
real Team Foundation Build or my "imitation Team Foundation Build" -- you need a
way to build the WSPs without referring to referenced assemblies using relative
paths.

As I first mentioned in my previous post, relative paths work just fine when
compiling from within Visual Studio or using MSBuild from the command line.
However, they don't work at all when queuing the builds through Team Foundation
Build.

The problem with relative paths is that Team Foundation Build uses a different
folder structure when compiling your projects. Specifically, it changes the
output folder for all compiled items to be under a new **Binaries** folder --
not the location specified in the project settings within Visual Studio.

In other words, if you refer to a referenced assembly using something like:

> ..\\..\\..\CoreServices\bin\\%BUILD\_CONFIGURATION%\Fabrikam.Demo.CoreServices.dll

then you will find that this works just fine when building through Visual Studio
-- or even when compiling using TFSBuild.proj from the command line (a.k.a. a
"Desktop Build"). However, if you then queue the build through Team Foundation
Server, you'll find your build fails because the referenced assembly was
actually output to a different folder.

If you dive into the log file for the build, you will find that Team Foundation
Build modifies the **OutDir** variable and sets it to something like:

> C:\Users\svc-build\AppData\Local\Temp\Demo\Daily Build - Main\Binaries\Debug\

So the trick to building WSPs with Team Foundation Build is to leverage the
**OutDir** variable instead of relying on relative paths to referenced
assemblies.

Here is the updated DDF file based on
[my earlier sample](/blog/jjameson/2009/09/28/sample-walkthrough-of-the-dr-dada-approach-to-sharepoint):

```
;
; This ddf specifies the structure of the .wsp solution cab file.
;
; HACK: OPTION EXPLICIT cannot be used when specifying a variable with the /D option,
; otherwise MakeCAB aborts with an error similar to the following:
;
;        ERROR: Option Explicit and variable not defined: OUT_DIR
;
;.OPTION EXPLICIT    ; Generate errors for undefined variables

.Set CabinetNameTemplate=Fabrikam.Demo.Publishing.wsp

; The following variable must be set when calling MakeCAB (using the /D option)
;.Define OUT_DIR=

.Set DiskDirectoryTemplate=CDROM    ; All cabinets go in a single directory
.Set CompressionType=MSZIP            ; All files are compressed in cabinet files
.Set UniqueFiles=ON
.Set Cabinet=ON
.Set DiskDirectory1=%OUT_DIR%\Package

DeploymentFiles\PackageFiles\manifest.xml

.Set SourceDir=%OUT_DIR%            ; Copy assemblies from %OUT_DIR% folder

Fabrikam.Demo.Publishing.dll
Fabrikam.Demo.CoreServices.dll

.Set SourceDir=                    ; Copy files relative to project folder

.Set DestinationDir=Fabrikam.Demo.Publishing.DefaultSiteConfiguration
DefaultSiteConfiguration\FeatureFiles\Feature.xml

.Set DestinationDir=Fabrikam.Demo.Publishing.Layouts
Layouts\FeatureFiles\Feature.xml
Layouts\FeatureFiles\ProvisionedFiles.xml

.Set DestinationDir=Fabrikam.Demo.Publishing.Layouts\MasterPages
Layouts\MasterPages\FabrikamMinimal.master

;.Set DestinationDir=Fabrikam.Demo.Publishing.Layouts\PageLayouts
;Layouts\PageLayouts\MinimalPage.aspx

.Set DestinationDir=Fabrikam.Demo.Publishing.Layouts\Images
Layouts\Images\FabrikamLogo_32x32.png

.Set DestinationDir=Fabrikam.Demo.Publishing.Layouts\Themes\Theme1
Layouts\Themes\Theme1\BreadcrumbBullet.gif
Layouts\Themes\Theme1\FauxColumn-Fixed-2Col.png
Layouts\Themes\Theme1\FauxColumn-Fixed-3Col.png
Layouts\Themes\Theme1\Fabrikam-Basic.css
Layouts\Themes\Theme1\Fabrikam-Core.css
Layouts\Themes\Theme1\Fabrikam-FixedLayout.css
Layouts\Themes\Theme1\Fabrikam-IE.css
Layouts\Themes\Theme1\Fabrikam-IE6.css
Layouts\Themes\Theme1\Fabrikam-QuirksMode.css
Layouts\Themes\Theme1\Tab-LeftSide.jpg
Layouts\Themes\Theme1\Tab-RightSide.jpg

.Set DestinationDir=ControlTemplates\Fabrikam\Demo\Publishing\Layouts
Layouts\Web\UI\WebControls\GlobalNavigation.ascx
Layouts\Web\UI\WebControls\StyleDeclarations.ascx

.Set DestinationDir=Layouts\Fabrikam
Layouts\MasterPages\FabrikamMinimal.master
```

Note how I've replaced the BUILD\_CONFIGURATION variable with the OUT\_DIR
variable. Not surprisingly, the OUT\_DIR variable in the DDF is specified
similar to how BUILD\_CONFIGURATION was previously specified when calling
makecab.exe. However, unlike the build configuration the OutDir variable will
likely contain spaces as well as a trailing slash (which makecab.exe apparently
doesn't like). Therefore we must quote the OutDir variable and append with "."
if a trailing slash is found.

Here is the corresponding update to the project file:

```XML
<PropertyGroup>
  <BuildDependsOn>
    $(BuildDependsOn);
    CreateSharePointSolutionPackage
  </BuildDependsOn>
  <QuotedOutDir>"$(OutDir)"</QuotedOutDir>
  <QuotedOutDir Condition="HasTrailingSlash($(OutDir))">"$(OutDir)."</QuotedOutDir>
</PropertyGroup>
<Target Name="CreateSharePointSolutionPackage" Inputs="@(None);@(Content);$(OutDir)$(TargetFileName);" Outputs="$(ProjectDir)$(OutDir)Package\Fabrikam.Demo.Publishing.wsp">
  <Message Text="Creating SharePoint solution package..." />
  <Exec Command="makecab /D OUT_DIR=$(QuotedOutDir) /F &quot;$(ProjectDir)DeploymentFiles\PackageFiles\wsp_structure.ddf&quot;" />
</Target>
```

With these changes, the SharePoint WSP is successfully built regardless of
whether it is compiled through Visual Studio, from the command line using
MSBuild and the TFSBuild.proj file, or as an automated build using a Team
Foundation Build server.
