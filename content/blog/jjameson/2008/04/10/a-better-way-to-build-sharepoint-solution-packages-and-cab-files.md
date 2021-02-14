---
title: "A Better Way to Build SharePoint Solution Packages (and CAB Files)"
date: 2008-04-10T11:28:00+08:00
excerpt: "Up until about an hour ago, I'd been using post-build events on my Visual Studio projects to create SharePoint solution packages (WSPs). However, while this worked reasonably well, this method always bothered me a little because the post-build events..."
draft: true
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "Core Development", "WSS v3"]
---

> **Note**
> 
>             This post originally appeared on my MSDN blog:  
>   
> 
> 
> [http://blogs.msdn.com/b/jjameson/archive/2008/04/10/a-better-way-to-build-sharepoint-solution-packages-and-cab-files.aspx](http://blogs.msdn.com/b/jjameson/archive/2008/04/10/a-better-way-to-build-sharepoint-solution-packages-and-cab-files.aspx)
> 
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.


Up until about an hour ago, I'd been using post-build events on my Visual Studio         projects to create SharePoint solution packages (WSPs). However, while this worked         reasonably well, this method always bothered me a little because the post-build         events run every time you build, regardless of whether the underlying files have         changed or not. In other words, using post-build events is just a "brute force"         method that builds the CAB (i.e. WSP) file with no dependency checking whatsoever.

Here is a sample of a post-build event used to create a SharePoint solution package:



    @echo Creating SharePoint solution package...
    makecab /F "$(ProjectDir)DeploymentFiles\ProductionDeployment\wsp_structure.ddf"



Converting this simple "makecab" command to an MSBuild target is easy -- just use         an `Exec` task:



    <Target Name="CreateSharePointSolutionPackage">
      <Message Text='Creating SharePoint solution package...' />
      <Exec Command='makecab /F "$(ProjectDir)DeploymentFiles\ProductionDeployment\wsp_structure.ddf"'
        WorkingDirectory='$(OutDir)' />
    </Target>



The only hiccup that tripped me up a little is the fact that I had to set the `WorkingDirectory` attribute to avoid         having to make any changes to the paths specified in the DDF file (since they were         previously specified using relative paths compatible with running <kbd>makecab.exe</kbd>         in the post-build event). Also, getting it to work without the `WorkingDirectory` attribute would have changed the location of         the generated CAB -- er, I mean WSP -- file, which is not what I wanted anyway.

At this point, however, note that I haven't really improved the situation at all         (i.e. the CAB/WSP file is created each time you build). To achieve the desired goal         (i.e. building the CAB/WSP file only when necessary), you need to specify `Inputs` and `Outputs`         on the `Target`. I then tested the         following to verify that I was on the right track:



    <Target Name="CreateSharePointSolutionPackage"
      Inputs="DeploymentFiles\ProductionDeployment\wsp_structure.ddf"
      Outputs='$(ProjectDir)$(OutDir)Package\Fabrikam.Project1.PublicationContentTypes.wsp'>
      <Message Text='Creating SharePoint solution package...' />
      <Exec Command='makecab /F "$(ProjectDir)DeploymentFiles\ProductionDeployment\wsp_structure.ddf"'
        WorkingDirectory='$(OutDir)' />
    </Target>



This worked (meaning, pressing <kbd>Ctrl+Shift+B</kbd> repeatedly resulted in essentially         no work for Visual Studio) but obviously it only checks for an updated DDF file         when deciding whether or not to build the target -- which is fundamentally incorrect         since the DDF may specify essentially any file in the project to include in the         solution package.

What I really wanted to specify in the `Inputs`         attribute is all of the items in the project, since any of them may be included         in the DDF and consequently in the SharePoint solution package. In other words,         I wanted to specify all of the following:



    <ItemGroup>
      <Reference Include="...">
    </ItemGroup
    <ItemGroup>
      <Compile Include="...">
    </ItemGroup
    <ItemGroup>
      <None Include="...">
    </ItemGroup
    <ItemGroup>
      <Content Include="...">
    </ItemGroup
    <ItemGroup>
      <ProjectReference Include="...">
    </ItemGroup>



Cliff Hudson, an SDE on the Visual Studio Platform team, was kind enough to point         me in the right direction. [Thanks for the tip, Cliff.]


> **Update (2008-04-14)**
> 
> 
> When I originally wrote this blog post last Friday, I specified the following:
> 
> 
> > `Inputs="@(Compile);@(None);@(Content);@(ProjectReference);"`
> 
> 
> However, early this morning I discovered that there are a couple of rare scenarios                 where the WSP/CAB file is not rebuilt when specifying those inputs -- even though                 the actual assembly is recompiled.
> 
> The first scenario is due to project references. If you dive deep into Microsoft.Common.targets,                 you'll find targets like **SplitProjectReferencesByType**, **ResolveProjectReferences**,                 **ResolveVCProjectReferences**, and **ResolveReferences**.                 These handle the "expansion" of the project references to determine the actual list                 of dependent files corresponding to the project references. Not expanding the project                 references in the list of inputs for `CreateSharePointSolutionPackage`is usually benign, unless the referenced assembly is actually deployed                 within the "referencing" project. For example, if ProjectA actually builds the WSP/CAB                 file, but also includes the assembly from ProjectB, then a minor change in ProjectB                 would first rebuild ProjectB, and then ProjectA, but would not rebuild the corresponding                 WSP/CAB file.
> 
> The second scenario that I discovered is when your project specifies embedded resources.                 Unfortunately, the project that I started with last Friday did not include embedded                 resources and consequently I did not notice that additional item group until this                 morning when I was hunting around in Microsoft.Common.targets.
> 
> The updated inputs below ensure that the WSP/CAB is built whenever the project assembly                 is built, by replacing `@(ProjectReference)`                 with `$(OutDir)$(TargetFileName)`                 -- which also made `@(Compile)` superfluous.                 In other words, it handles scenarios where the only change is to an embedded resource                 or when a dependent assembly specified using a project reference is updated.


Here is the updated MSBuild target with the correct inputs specified:



    <Target Name="CreateSharePointSolutionPackage"
      Inputs="@(None);@(Content);$(OutDir)$(TargetFileName);"
      Outputs='$(ProjectDir)$(OutDir)Package\Fabrikam.Project1.PublicationContentTypes.wsp'>
      <Message Text='Creating SharePoint solution package...' />
      <Exec Command='makecab /F "$(ProjectDir)DeploymentFiles\ProductionDeployment\wsp_structure.ddf"'
        WorkingDirectory='$(OutDir)' />
    </Target>



Replacing the post-build event with this target in the project file achieves the         desired goal of creating or updating the SharePoint solution only when necessary         (i.e. when one or more of the files comprising the CAB/WSP is updated).

The final piece is to "wire in" the custom target. There are a couple of ways to         do this:

1. Append `CreateSharePointSolutionPackage`
            to the `DefaultTargets` attribute
            for the project, or
2. Add it as a dependency of the default `Build`
            target, using [the technique](http://blogs.msdn.com/msbuild/archive/2006/02/10/528822.aspx) described by Neil Enns (a.k.a. "Demo Boy") on the [MSBuild Team blog](http://blogs.msdn.com/msbuild).


The second option seems a little more elegant than the first, and hence is what         I chose to use:



    <PropertyGroup>
      <BuildDependsOn>
        $(BuildDependsOn);
        CreateSharePointSolutionPackage
      </BuildDependsOn>
    </PropertyGroup>



Building SharePoint solution packages by modifying the MSBuild targets certainly         isn't a new concept. It has been over a year since I first read Andrew Connell's         [blog post](http://www.andrewconnell.com/blog/articles/UsingVisualStudioAndMsBuildToCreateWssSolutions.aspx) on this. However, there were a couple of things that bothered         me about Andrew's approach; the most important being that I didn't see any compelling         reasons to switch from using post-build events. With true dependency checking to         avoid superfluous calls to <kbd>makecab.exe</kbd>, this is obviously no longer the         case.

