---
title: Bypassing a Gated Check-in in TFS 2010
date: 2010-12-03T07:32:00-07:00
excerpt:
  Yesterday someone contacted me about my earlier post on Incrementing the
  Assembly Version for Each Build in TFS 2010 , because after following the
  steps I provided, he encountered a problem due to the fact that he had
  previously configured a gated check...
aliases:
  [
    "/blog/jjameson/archive/2010/12/02/bypassing-a-gated-check-in-in-tfs-2010.aspx",
    "/blog/jjameson/archive/2010/12/03/bypassing-a-gated-check-in-in-tfs-2010.aspx",
  ]
categories: ["Development"]
tags: ["TFS"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/12/03/bypassing-a-gated-check-in-in-tfs-2010.aspx"
---

Yesterday someone contacted me about my earlier post on
[Incrementing the Assembly Version for Each Build in TFS 2010](/blog/jjameson/2010/11/29/incrementing-the-assembly-version-for-each-build-in-tfs-2010),
because after following the steps I provided, he encountered a problem due to
the fact that he had previously configured a gated check-in build that included
the folder containing the AssemblyVersionInfo files.

The problem is that after you create a gated check-in build definition, the {{<
kbd "tf.exe checkin" >}} command does not actually check-in your changes, but
rather shelves your changes until the gated check-in validates the changeset (at
which point, the files are checked in on your behalf). In this case, the {{< kbd
"tf.exe checkin" >}} command does not return 0 and the original build (that was
attempting to check-in the AssemblyVersionInfo files) consequently fails.

Fortunately, the Visual Studio folks have provided a way to bypass gated
check-ins (provided a user has been granted permission to do so). Therefore, to
avoid the issue when incrementing the assembly version for each build, you first
need to grant the appropriate permission to the serice account used to perform
your builds.

To grant permission to bypass a gated check-in:

1. In the **Team Explorer** window, expand the appropriate project, right-click
   **Builds**, and then click **Security...**
1. In the **{Project} Security** window, in the **Add users and groups**
   section, click **Windows User or Group**, and then click **Add...**
1. In the **Select Users, Computers, or Groups** window, type the name of the
   service account used for TFS builds (e.g. **TECHTOOLBOX\svc-build**), and
   then click **OK**.
1. In the **{Project} Security** window, in the list of permissions, for the
   **Override check-in validation by build** permission, click the checkbox in
   the **Allow** column, and then click **OK**.

Then you simply need to modify the IncrementAssemblyVersion.proj file (described
in my previous post) to specify the {{< kbd "/bypass" >}} option with the {{<
kbd "tf.exe checkin" >}} command.

```XML
<Exec
  WorkingDirectory="$(BuildProjectFolderPath)"
  Command="$(TeamFoundationVersionControlTool) checkin /bypass /override:&quot;Check-in from automated build&quot; /comment:&quot;Increment assembly version ($(IncrementedAssemblyVersion)) $(NoCICheckinComment)&quot; AssemblyVersionInfo.txt AssemblyVersionInfo.cs"/>
</Project>
```

Once you have made these changes, the build that increments the assembly version
will run without issue.

For more information on gated check-ins, refer to the following:

{{< reference title="Define a Gated Check-In Build to Validate Changes"
linkHref="http://msdn.microsoft.com/en-us/library/dd787631.aspx" >}}

In case you are wondering how I configured the gated check-in build definition,
here are the settings I used. If a setting is not listed in the following table,
it means the default is used.

<div class="d-md-none">
  <a href='{{< relref "resources/table-1-popout" >}}' target="_blank">Table 1 - Build Definition: <strong>Gated Check-in - Main</strong></a>
  <i class="bi bi-arrow-up-right-square"></i>
  <p>(Insufficient width to show table content here.)</p>
</div>
<div class="d-none d-md-block">
  {{< include-html "resources/table-1.html" >}}
</div>
