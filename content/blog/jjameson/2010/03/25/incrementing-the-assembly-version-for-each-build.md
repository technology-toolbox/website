---
title: "Incrementing the Assembly Version for Each Build"
date: 2010-03-25T04:59:00-06:00
excerpt: "Last summer I wrote a post about best practices for .NET assembly versioning and made the following statement: 
 The AssemblyFileVersionAttribute should be incremented automatically as part of the build process. 
 In the comments for that post, someone..."
aliases: ["/blog/jjameson/archive/2010/03/24/incrementing-the-assembly-version-for-each-build.aspx", "/blog/jjameson/archive/2010/03/25/incrementing-the-assembly-version-for-each-build.aspx"]
draft: true
categories: ["My System", "Development"]
tags: ["My System", "Core Development", "TFS"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/03/25/incrementing-the-assembly-version-for-each-build.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/03/25/incrementing-the-assembly-version-for-each-build.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Last summer I wrote a post about
[best practices for .NET assembly versioning](/blog/jjameson/2009/04/03/best-practices-for-net-assembly-versioning)
and made the following statement:

{{< blockquote "font-italic" >}}

The
[AssemblyFileVersionAttribute](http://msdn.microsoft.com/en-us/library/system.reflection.assemblyfileversionattribute%28VS.71%29.aspx)
should be incremented automatically as part of the build process.

{{< /blockquote >}}

In the comments for that post, someone asked exactly what I meant by that --
specifically if I used something in the pre-build event to increment the
assembly version.

Here was my response:

{{< blockquote "font-italic" >}}

While you certainly \*could\* increment the AssemblyFileVersionAttribute in a
pre-build event, I definitely don't recommend it. Doing so would cause the
version to increment each and every time \*any\* member of the Development team
builds the solution.

I suppose you could simply tell developers not to check-in the updated
AssemblyVersionInfo.cs file, but there are definitely better ways to accomplish
the desired outcome.

Rather, I recommend incrementing the AssemblyFileVersionAttribute as part of
your automated build process. In other words, each time an "official" build is
created on the Build Server, the AssemblyVersionInfo.cs file is automatically
checked out from source control, incremented, and checked back in.

Obviously, the actual implementation of this process will vary depending on your
particular toolset. For example, if you are using Team Foundation Server, you
can setup a custom task that increments the AssemblyFileVersionAttribute as part
of the build. Several people have already blogged about the details of this for
TFS. If you just bing "TFS increment build" you should get some good hits within
the first page of search results. In particular, make sure you read Buck Hodges
blog entry if you are using continuous integration.

{{< /blockquote >}}

Well, I probably should have blogged long ago about the specific process that I
use for incrementing the assembly version, but you know what they say: "better
late than never" ;-)

Note that this implementation has some specifics to Team Foundation Server, but
I imagine you could tweak this fairly easily if you are using some other
configuration management system and build process.

> **Update (2010-11-29)**
>
> This post was originally created for TFS 2005/2008. Refer to the following if
> you are using TFS 2010:
>
> {{< reference title="Incrementing the Assembly Version for Each Build in TFS 2010" linkHref="/blog/jjameson/2010/11/29/incrementing-the-assembly-version-for-each-build-in-tfs-2010" linkText="http://blogs.msdn.com/b/jjameson/archive/2010/11/29/incrementing-the-assembly-version-for-each-build-in-tfs-2010.aspx" >}}

Unfortunately, there's no out-of-the-box task in the current version of MSBuild
that increments an assembly version. However, you can write your own with just a
few lines of code (there are a number of samples out there if you search for
them), or -- as I prefer -- you can just use the one from the
[MSBuild Community Tasks Project](http://msbuildtasks.tigris.org/). [There are
quite a few other custom tasks in this package that you may find useful in
addition to the Version task that I cover in this post.]

After downloading and installing the custom tasks, import them into your
TFSBuild.proj file by adding the following line just below the `<Project>`
element:

```
  <Import Project="$(MSBuildExtensionsPath)\MSBuildCommunityTasks\MSBuild.Community.Tasks.Targets"/>
```

Note that there may be times when we want to build the solution without
incrementing the assembly version; for example, to test changes to TFSBuild.proj
on a local development environment (as opposed to checking in the changes and
"testing" the changes on the build server). Therefore, let's allow a custom
property to be specified that will skip the process of incrementing the assembly
version. Inside the `<PropertyGroup>` element, add the following:

```
    <!-- Set this property to true to build without incrementing the assembly version. -->
    <SkipIncrementAssemblyVersion
      Condition=" '$(SkipIncrementAssemblyVersion)' == '' " >false</SkipIncrementAssemblyVersion>
```

Here is an example of specifying this property as a command-line option:

{{< console-block-start >}}

msbuild TFSBuild.proj /property:SkipIncrementAssemblyVersion=true

{{< console-block-end >}}

Next, add a property so that we can use the TFS command-line utility to checkout
the assembly version files and subsequently check them back in:

```
  <PropertyGroup>
    <TeamFoundationVersionControlTool>&quot;$(TeamBuildRefPath)\..\tf.exe&quot;</TeamFoundationVersionControlTool>
  </PropertyGroup>
```

> **Update (2010-05-05)**
>
> Note that the path to the TFS command-line utility
> [has changed for a TFS 2010 build server](/blog/jjameson/2010/05/05/updated-path-to-tf-exe-for-tfs-2010-builds).
>
> To use the same technique on a TFS 2010 build server, specify the following
> instead:
>
> ```
>   <PropertyGroup>
>     <TeamFoundationVersionControlTool>&quot;$(VS100COMNTOOLS)..\IDE\tf.exe&quot;</TeamFoundationVersionControlTool>
>   </PropertyGroup>
> ```

One of the things that I've struggled with in the past is that "Desktop Builds"
and "Team Builds" behave quite differently in certain areas. For example, when
performing a Desktop Build (i.e. running msbuild on TFSBuild.proj from a command
prompt), the **SolutionRoot** and **BuildProjectFolderPath** properties for my
sample Fabrikam solution are set as follows:

- **SolutionRoot:** C:\NotBackedUp\Fabrikam\Demo
- **BuildProjectFolderPath:** C:\NotBackedUp\Fabrikam\Demo\Main\Source

However, when performing a Team Build (i.e. when a build is queued or scheduled
on a build server), the properties are set as follows:

- **SolutionRoot:** C:\Users\svc-build\AppData\Local\Temp\Demo\Automated Build -
  Main\Sources
- **BuildProjectFolderPath:** $/Demo/Main/Source

Consequently, define a new property (**SolutionWorkingDirectory**) and
condititionally set it to the expected location:

```
  <!-- HACK: The values of $(SolutionRoot) and $(BuildProjectFolderPath) vary
  significantly between desktop builds and builds started using Team
  Foundation Build (in other words, builds queued on TFS build agents).

  For desktop builds:

  SolutionRoot = C:\NotBackedUp\Fabrikam\Demo
  BuildProjectFolderPath = C:\NotBackedUp\Fabrikam\Demo\Main\Source

  For builds performed using Team Foundation Build:

  SolutionRoot = C:\Users\svc-build\AppData\Local\Temp\Demo\Automated Build - Main\Sources
  BuildProjectFolderPath = $/Demo/Main/Source

  In order to update files (i.e. check out and subseqeuntly check-in) in the solution
  regardless of which build type is currently running, conditionally  set the
  "SolutionWorkingDirectory" property.
  -->
  <PropertyGroup Condition=" '$(IsDesktopBuild)' == 'true' ">
    <SolutionWorkingDirectory>$(BuildProjectFolderPath)</SolutionWorkingDirectory>
  </PropertyGroup>
  <PropertyGroup Condition=" '$(IsDesktopBuild)' != 'true' ">
    <SolutionWorkingDirectory>$(SolutionRoot)\Source</SolutionWorkingDirectory>
  </PropertyGroup>
```

Next, override the **AfterGet** target to checkout the version files from TFS,
and subsequently update the assembly version:

```
  <Target Name="AfterGet">
    <CallTarget Targets="CheckOutVersionFilesFromSourceControl"/>
    <CallTarget Targets="UpdateVersionFilesInSourceControl"/>
  </Target>
```

Here is the custom target to checkout the version files. Note that this target
is skipped if the **SkipIncrementAssemblyVersion** property is set to true.

```
  <Target Name="CheckOutVersionFilesFromSourceControl"
    Condition=" '$(SkipIncrementAssemblyVersion)' != 'true' ">
    <Message Importance="high"
      Text="Checking out version files from source control..." />

    <Message Importance="high"
      Text="SolutionWorkingDirectory: $(SolutionWorkingDirectory)" />

    <Exec
      WorkingDirectory="$(SolutionWorkingDirectory)"
      Command="$(TeamFoundationVersionControlTool) checkout AssemblyVersionInfo.txt AssemblyVersionInfo.cs"/>

  </Target>
```

Note that AssemblyVersionInfo.txt is just a simple text file used by the custom
Version task from the MSBuild Community Tasks Project and only contains a simple
version string. For example:

```
1.0.38.0
```

The AssemblyVersionInfo.cs file is generated by the custom Version task. For
example:

```
//------------------------------------------------------------------------------
// <auto-generated>
//     This code was generated by a tool.
//     Runtime Version:2.0.50727.4200
//
//     Changes to this file may cause incorrect behavior and will be lost if
//     the code is regenerated.
// </auto-generated>
//------------------------------------------------------------------------------

using System;
using System.Reflection;
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;

[assembly: AssemblyFileVersion("1.0.38.0")]
```

Here is the custom target to update the version files in TFS. Like the
`CheckOutVersionFilesFromSourceControl` target, the
`UpdateVersionFilesInSourceControl` target is skipped if the
**SkipIncrementAssemblyVersion** property is set to true.

```
  <Target Name="UpdateVersionFilesInSourceControl"
    Condition=" '$(SkipIncrementAssemblyVersion)' != 'true' ">
    <Message Importance="high"
      Text="Updating version files in source control..." />

    <Message Importance="high"
      Text="SolutionWorkingDirectory: $(SolutionWorkingDirectory)" />

    <Copy
      Condition=" '$(SkipIncrementAssemblyVersion)' != 'true' and '$(IsDesktopBuild)' != 'true' "
      SourceFiles="$(SolutionRoot)\..\BuildType\AssemblyVersionInfo.txt"
      DestinationFiles="$(SolutionWorkingDirectory)\AssemblyVersionInfo.txt"/>

    <AssemblyInfo
      Condition=" '$(SkipIncrementAssemblyVersion)' != 'true' "
      CodeLanguage="CS"
      OutputFile="$(SolutionWorkingDirectory)\AssemblyVersionInfo.cs"
      AssemblyFileVersion="$(Major).$(Minor).$(Build).$(Revision)" />

    <Exec
      WorkingDirectory="$(SolutionWorkingDirectory)"
      Command="$(TeamFoundationVersionControlTool) checkin
 /override:&quot;Check-in from automated build&quot;
 /comment:&quot;Increment assembly version ($(BuildNumber)) $(NoCICheckinComment)&quot;
 AssemblyVersionInfo.txt AssemblyVersionInfo.cs"/>
  </Target>
```

Note that for a Team Build, we need to copy the AssemblyVersionInfo.txt file
from the BuildType folder into the solution folder. This allows the same process
to be used for Desktop Builds and Team Builds. Also note that
`$(NoCICheckinComment)` is specified when checking in the files from the build
(as I mentioned in my earlier comment, see Buck Hodges'
[blog post](http://blogs.msdn.com/buckh/archive/2007/07/27/tfs-2008-how-to-check-in-without-triggering-a-build-when-using-continuous-integration.aspx)
for more details on this).

Finally, use the **BuildNumberOverrideTarget** and a custom target to actually
increment the assembly version using the custom Version task and set the
**BuildNumber** property accordingly. Note that if
**SkipIncrementAssemblyVersion** is set to true, the assembly version is not
incremented and the **BuildNumber** property is set to whatever is currently
specified in AssemblyVersionInfo.txt.

```
  <Target Name="BuildNumberOverrideTarget">
    <CallTarget Targets="SetBuildNumber"/>
  </Target>

  <Target Name="SetBuildNumber">
    <Attrib Files="AssemblyVersionInfo.txt" ReadOnly="false"/>

    <Version
      Condition=" '$(SkipIncrementAssemblyVersion)' == 'true' "
      VersionFile="AssemblyVersionInfo.txt"
      BuildType="None"
      RevisionType="None">
      <Output TaskParameter="Major" PropertyName="Major" />
      <Output TaskParameter="Minor" PropertyName="Minor" />
      <Output TaskParameter="Build" PropertyName="Build" />
      <Output TaskParameter="Revision" PropertyName="Revision" />
    </Version>

    <Version
      Condition=" '$(SkipIncrementAssemblyVersion)' != 'true' "
      VersionFile="AssemblyVersionInfo.txt"
      BuildType="Increment"
      RevisionType="None">
      <Output TaskParameter="Major" PropertyName="Major" />
      <Output TaskParameter="Minor" PropertyName="Minor" />
      <Output TaskParameter="Build" PropertyName="Build" />
      <Output TaskParameter="Revision" PropertyName="Revision" />
    </Version>

    <PropertyGroup>
      <BuildNumber>$(Major).$(Minor).$(Build).$(Revision)</BuildNumber>
    </PropertyGroup>
    <Message Importance="high"
      Text="Build number set to &quot;$(BuildNumber)&quot;" />
  </Target>
```

Note that in the example above, the `BuildType` attribute on the `<Version>`
element is set to `"Increment"` when the assembly version should be incremented
(thus generating build numbers like 1.0.37.0, 1.0.38.0, etc.). This is what I
recommend for the **Main** branch.

For the **QFE** branch, I recommend changing the `BuildType` attribute to
`"None"` and the `RevisionType` attribute to `"Increment"` (to generate build
numbers like 1.0.38.1, 1.0.38.2, etc.).

Refer to one of my previous posts more information on
[shared assembly files in Visual Studio projects](/blog/jjameson/2009/04/03/shared-assembly-info-in-visual-studio-projects).
