---
title: "Branching for a Release in Team Foundation Server"
date: 2010-12-03T06:05:00+08:00
excerpt: "In my previous post , I mentioned that one of the recurring tasks I create in TFS each time I start a new iteration on a project is something like \"Create branch for Sprint-10\" (the iteration specified in the title of the work item obviously varies each..."
draft: true
categories: ["My System", "Development"]
tags: ["My System", "TFS"]
---

> **Note**
> 
>             This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2010/12/03/branching-for-a-release-in-team-foundation-server.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/12/03/branching-for-a-release-in-team-foundation-server.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

In [my previous post](/blog/jjameson/2010/12/03/recurring-tasks-in-team-foundation-server), I mentioned that one of the recurring tasks I create in TFS each time I start a new iteration on a project is something like "Create branch for Sprint-10" (the iteration specified in the title of the work item obviously varies each time).

This work item serves a couple of purposes:

- It reminds me to ensure the code is actually branched for a release (something I'm
  not likely to forget, but still...)
- It provides a work item to associate with the actual changesets used to branch the
  code

Since one of the things I need to do today on my current project is branch the code, I thought this would be a good opportunity to share the process that I use.

First, I should briefly explain the structure of the TFS project (which I'll simply refer to as "FabrikamPortal"). Here is what the **Source Control Explorer
**window currently looks like for the project:

- FabrikamPortal
  - Dev
    - ...
  - Main
    - Code
    - Docs
  - Release
    - v1.0
      - Sprint-1
        - QFE
        - RTM
        - ServicePack
      - Sprint-2
        - QFE
        - RTM
        - ServicePack
      - ...

In general, this structure follows the "Advanced Branch Plan" specified in the TFS Branching Guide that was put together by the ALM Rangers. If you haven't seen the latest version (for TFS 2010), I definitely recommend taking a look at it:

<cite>Visual Studio TFS Branching Guide 2010 </cite>
[http://tfsbranchingguideiii.codeplex.com/](http://tfsbranchingguideiii.codeplex.com/)

Note that for "Sprint-10" we are actually deploying "v2.0" of the portal (a "major" upgrade to support regional deployments in other countries, i.e. Argentina, Mexico, and Spain). Therefore, I need to branch $/FabrikamPortal/Main to $/FabrikamPortal/Release/v2.0/Sprint-10. [Honestly, I sort of wish we referred to the current iteration as "v2.0\Sprint-1" from the beginning of the sprint, but that's not the way it worked out. Hence, I'm sticking with the "Sprint-10" moniker for the time being.]

Also note that it turns out we don't really need both a "Service Pack" branch as well as "Hot Fix" branch (or, "QFE" as I tend to call it) -- since we start a new sprint every 6-8 weeks (with corresponding deployments to Production).

In hindsight, at the end of Sprint-1, I should have created just the **ServicePack** branch and the **RTM** branch (i.e. the equivalent of the "Standard Branch Plan") -- but honestly, at that time I didn't really know what to expect with regards to subsequent deployments to PROD for this particular project. Consequently, I elected to use the branching model that provided the most flexibility (but also the one that requires more work to merge changes from the QFE branch eventually back to Main).

> **Important**
> 
>             Going forward, I'm going to switch from the "Advanced Branch Plan" to the "Standard
>             Branch Plan" (since -- at least for this particular project -- we don't need both
>             a **ServicePack** branch as well as a **QFE** branch).

Note that build 2.0.371.0 is the version that has been approved for this release. When branching code for a release, I always branch from **Main **using a specific changeset or label (corresponding to the version approved for release). By branching from a specific changeset or label, it doesn't really matter when you create the branch. In other words, you don't have to worry about whether any changes have been checked in on **Main **after the build that is considered "golden." Also note that you will likely decide to branch for a release before the final version has been determined (so that some members of the Development team can keep working on **Main **while others focus on fixing bugs in the release branch).

In this particular case, build 2.0.371.0 corresponds to changeset 302281, so I'll use that changeset when creating the release branch.

Branching for a release (with the "Standard Branch Plan") is comprised of three logical steps:

1. Create a new "Service Pack" branch from the "Main" branch
2. Create the "RTM" branch from the new "Service Pack" branch
3. Changing the "Service Pack" branch to increment the Revision portion of the assembly
   number instead of the Build (e.g. 2.0.371.1).

> **Note**
> 
>             If you are using the "Advanced Branch Plan" (as I was before), then branching for
>             a release is comprised of *four* logical steps (because you would first branch
>             "Service Pack" to "QFE" and then "QFE" to "RTM"). Also note that you would increment
>             the Revision portion of the assembly number on the "QFE" branch instead of the "Service
>             Pack" branch.

To create a new "Service Pack" branch from the "Main" branch:

1. In **Source Control Explorer**, right-click the **Main **
   branch for the project, point to **Branching and Merging**, and then
   click **Branch**.
2. In the **Branch **window:
   1. In the **Target **box, type the path for the new "Service Pack" branch
      (e.g. **$/FabrikamPortal/Release/v2.0/Sprint-10/ServicePack**).
   2. In the **Branch from version **section, in the **By **
      dropdown list, select **Changeset **and then type the corresponding
      changeset number (e.g. **302281**).
   3. Clear the **Download the target item to your workspace **checkbox.
   4. Click **OK**.
3. Wait for the branching operation to complete and then in **Source Control
   Explorer**, right-click the new branch (e.g. **$/FabrikamPortal/Release/v2.0/Sprint-10/ServicePack**),
   and then click **Check In Pending Changes...**
4. In the **Check In - Source Files **window:
   1. In the **Source Files **channel, type a descriptive comment in the
      **Comment **box (e.g. **Branch changeset 302281 (build 2.0.371.0)
      from $/FabrikamPortal/Main to $/FabrikamPortal/Release/v2.0/Sprint-10/ServicePack**)
      and verify the list of files selected to check in.
   2. In the **Work Items **channel, select the corresponding work item (e.g.
      **Create branch for Sprint-10 release**), and set the **Check-in
      Action **to **Associate**.
   3. Click **Check In**. If necessary, override any policy failures (e.g.
      "The Code Analysis Policy requires files to be check in through Visual Studio with
      an open solution.")

> **Note**
> 
>             If it really bothers you to override check-in policy failures like the one noted
>             above, you can certainly download the new branch to your workspace and check in
>             the changes through Visual Studio with the solution open. However, at least in my
>             mind, this isn't worth the extra time it requires when simply branching the code
>             for a release.

To create the "RTM" branch from the new "Service Pack" branch:

1. In **Source Control Explorer**, right-click the new "Service Pack"
   branch created using the steps in the previous section, point to **Branching and
   Merging**, and then click **Branch**.
2. In the **Branch **window:
   1. In the **Target **box, type the path for the new "RTM" branch (e.g.
      **$/FabrikamPortal/Release/v2.0/Sprint-10/RTM**).
   2. In the **Branch from version **section, ensure that **Latest
      Version **is specified.
   3. Clear the **Download the target item to your workspace **checkbox.
   4. Click **OK**.
3. Wait for the branching operation to complete and then in **Source Control
   Explorer**, right-click the new branch (e.g. **$/FabrikamPortal/Release/v2.0/Sprint-10/RTM**),
   and then click **Check In Pending Changes...**
4. In the **Check In - Source Files **window:
   1. In the **Source Files **channel, type a descriptive comment in the
      **Comment **box (e.g. **Branch $/FabrikamPortal/Release/v2.0/Sprint-10/ServicePack
      to $/FabrikamPortal/Release/v2.0/Sprint-10/RTM**) and verify the list
      of files selected to check in.
   2. In the **Work Items **channel, select the corresponding work item (e.g.
      **Create branch for Sprint-10 release**), and set the **Check-in
      Action **to **Associate**.
   3. Click **Check In**. If necessary, override any policy failures (e.g.
      "The Code Analysis Policy requires files to be check in through Visual Studio with
      an open solution.")

To increment the Revision portion of the assembly version on the "Service Pack" branch (note that this assumes you are using the process I described in an earlier post for [incrementing the assembly version for each build](/blog/jjameson/2010/11/29/incrementing-the-assembly-version-for-each-build-in-tfs-2010)):

1. In **Source Control Explorer**, in the new "Service Pack" branch, right-click
   the **IncrementAssemblyVersion.proj **or **TFSBuild.proj **
   file that is used to increment the assembly version, and then click **Get Latest
   Version**.

2. Next, double-click the file to open it in the editor.

3. Location the **&lt;Version&gt;** task used to increment the assembly
   version and change the `BuildType`
   to `"None"` and the `RevisionType` to `"Increment"`,
   as shown below:
   
   ```
   <Version
         VersionFile="$(BuildProjectFolderPath)\AssemblyVersionInfo.txt"
         BuildType="None"
         RevisionType="Increment">
   ```

4. Save the changes to the file.

5. Right-click the updated MSBuild file and click **Check In Pending Changes...**

6. In the **Check In - Source Files **window:
   
   1. In the **Source Files **channel, type a descriptive comment in the
      **Comment **box (e.g. **Increment revision portion of assembly version
      on ServicePack branch (instead of the build number)**).
   2. In the **Work Items **channel, select the corresponding work item (e.g.
      **Create branch for Sprint-10 release**), and set the **Check-in
      Action **to **Resolve**.
   3. Click **Check In**. If necessary, override any policy failures (e.g.
      "The Code Analysis Policy requires files to be check in through Visual Studio with
      an open solution.")

> **Tip**
> 
>             Even though it probably seems silly to many people, I always compare my changes
>             on edited files before I click the **Check In **button -- even when
>             making trivial changes like the one described above. It doesn't take but a few seconds,
>             and it helps ensure I don't accidentally check in some unexpected changes.

