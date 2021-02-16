---
title: "Bypassing a Gated Check-in in TFS 2010"
date: 2010-12-03T00:32:00-07:00
excerpt: "Yesterday someone contacted me about my earlier post on Incrementing the Assembly Version for Each Build in TFS 2010 , because after following the steps I provided, he encountered a problem due to the fact that he had previously configured a gated check..."
draft: true
categories: ["Development"]
tags: ["TFS"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/12/03/bypassing-a-gated-check-in-in-tfs-2010.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/12/03/bypassing-a-gated-check-in-in-tfs-2010.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

Yesterday someone contacted me about my earlier post on [Incrementing the Assembly Version for Each Build in TFS 2010](/blog/jjameson/2010/11/29/incrementing-the-assembly-version-for-each-build-in-tfs-2010), because after following the steps I provided, he encountered a problem due to the fact that he had previously configured a gated check-in build that included the folder containing the AssemblyVersionInfo files.

The problem is that after you create a gated check-in build definition, the <samp>tf.exe checkin</samp> command does not actually check-in your changes, but rather shelves your changes until the gated check-in validates the changeset (at which point, the files are checked in on your behalf). In this case, the <samp>tf.exe checkin</samp> command does not return 0 and the original build (that was attempting to check-in the AssemblyVersionInfo files) consequently fails.

Fortunately, the Visual Studio folks have provided a way to bypass gated check-ins (provided a user has been granted permission to do so). Therefore, to avoid the issue when incrementing the assembly version for each build, you first need to grant the appropriate permission to the serice account used to perform your builds.

To grant permission to bypass a gated check-in:

1. In the **Team Explorer** window, expand the appropriate project, right-click **Builds**, and then click **Security...**
2. In the **{Project} Security** window, in the **Add users and groups** section, click **Windows User or Group**, and then click **Add...**
3. In the **Select Users, Computers, or Groups** window, type the name of the service account used for TFS builds (e.g. **TECHTOOLBOX\svc-build**), and then click **OK**.
4. In the **{Project} Security** window, in the list of permissions, for the **Override check-in validation by build** permission, click the checkbox in the **Allow** column, and then click **OK**.

Then you simply need to modify the IncrementAssemblyVersion.proj file (described in my previous post) to specify the <samp>/bypass</samp> option with the <samp>tf.exe checkin</samp> command.

```
<Exec
  WorkingDirectory="$(BuildProjectFolderPath)"
  Command="$(TeamFoundationVersionControlTool) checkin /bypass /override:&quot;Check-in from automated build&quot; /comment:&quot;Increment assembly version ($(IncrementedAssemblyVersion)) $(NoCICheckinComment)&quot; AssemblyVersionInfo.txt AssemblyVersionInfo.cs"/>
</Project>
```

Once you have made these changes, the build that increments the assembly version will run without issue.

For more information on gated check-ins, refer to the following:

{{< reference title="Define a Gated Check-In Build to Validate Changes" linkHref="http://msdn.microsoft.com/en-us/library/dd787631.aspx" >}}

In case you are wondering how I configured the gated check-in build definition, here are the settings I used. If a setting is not listed in the following table, it means the default is used.

**Build Definition: "Gated Check-in - Main"**

| Section | Property | Value |
| --- | --- | --- |
| General | Build definition name | Gated Check-in - Main |
| Trigger | Gated Check-in - accept check-ins only if the submitted changes merge and build successfully | (selected) |
| Workspace | Source Control Folder<br>Build Agent Folder | $/foobar2010/Main<br>$(SourceDir) |
| Build Defaults | This build copies output files to a drop folder | (not selected) |
| Process | Build process template: | DefaultTemplate.xaml |
|   | Build process parameters: |   |
|   | Items to Build<ul><li>Solutions/Projects</li>

<li>Configurations</li></ul> | <br><ul><li>$/foobar2010/Main/Source/TechnologyToolbox.Foobar.sln</li>
<li>Release - Any CPU</li></ul> |
|   | Clean Workspace | False |
|   | Perform Code Analysis | Never |
|   | Source And Symbol Server Settings<ul><li>Index Sources</li></ul> | <br>False |
|   | Agent Settings<ul><li>Maximum Agent Execution Time</li></ul> | <br>00:15:00 |
|   | Copy Outputs to Drop Folder | False |
|   | Create Work Item on Failure | False |
|   | Label Sources | False |


