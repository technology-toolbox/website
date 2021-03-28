---
title: "The \"Copy Local\" Bug in Visual Studio"
date: 2009-11-18T04:21:00-07:00
excerpt:
  "If you've ever worked with me on a Microsoft Office SharePoint Server (MOSS)
  2007 project -- or if you've read my Sample Walkthrough of the DR.DADA
  Approach to SharePoint -- then you've probably seen the following comment:
  Note: Referenced assemblies..."
aliases:
  [
    "/blog/jjameson/archive/2009/11/17/the-copy-local-bug-in-visual-studio.aspx",
    "/blog/jjameson/archive/2009/11/18/the-copy-local-bug-in-visual-studio.aspx",
  ]
draft: true
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "Core Development", "WSS v3", "Visual Studio"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/11/18/the-copy-local-bug-in-visual-studio.aspx"
---

If you've ever worked with me on a Microsoft Office SharePoint Server (MOSS)
2007 project -- or if you've read my
[Sample Walkthrough of the DR.DADA Approach to SharePoint](/blog/jjameson/2009/09/28/sample-walkthrough-of-the-dr-dada-approach-to-sharepoint)
-- then you've probably seen the following comment:

{{< div-block "fst-italic" >}}

> Note: Referenced assemblies must be specified with a path corresponding to the
> build configuration. If the path is not specified to the referenced assembly,
> then the build works fine as long as the referenced assembly is not in the
> GAC.
>
> However, when the referenced assembly is in the GAC (i.e. after a deployment)
> then MakeCAB will not be able to find the referenced assembly (since it is no
> longer copied to the current project's bin\Debug or bin\Release folder).

{{< /div-block >}}

It turns out that I was using a rather elaborate workaround for a problem that
is actually much easier to solve.

To workaround the "Copy Local" bug and force a referenced assembly to always be
copied to the output folder (regardless of whether the referenced assembly is in
the GAC):

1. In the **Solution Explorer** window in Visual Studio, expand the
   **References** folder for the project and then select the referenced
   assembly.
1. In the **Properties** window, change the value of **Copy Local** to
   **False**, and then change it back to **True**.

Following these two simple steps explicitly adds `<Private>True</Private>` to
the project file, as shown in the following example:

```XML
<ProjectReference Include="..\CoreServices\CoreServices.csproj">
  <Project>{01C58D27-9818-45D6-A0B6-8EF765CA9397}</Project>
  <Name>CoreServices %28CoreServices\CoreServices%29</Name>
  <Private>True</Private>
</ProjectReference>
```

When you add a referenced assembly in Visual Studio using a project reference,
**Copy Local** defaults to **True**, but Visual Studio doesn't explicitly state
this in the MSBuild project file. Toggling the value of **Copy Local** forces
this element to be added to the project file and consequently you no longer need
any hacks to reference the assembly in its original output folder.

At this point, you might be wondering why do I bring this up after all this
time? After all, hasn't the hack I came up with for building SharePoint Web
Solution Packages (WSPs) been working for several years? Well, yes, in most
cases it works just fine.

However, there's one fundamental problem that I only recently discovered back in
early October: you can't use Team Foundation Build to build the WSP when
specifying relative paths to assemblies in the DDF file.

I'll cover this in
[my next post](/blog/jjameson/2009/11/18/building-sharepoint-wsps-with-team-foundation-build).
