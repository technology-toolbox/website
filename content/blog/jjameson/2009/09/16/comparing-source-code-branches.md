---
title: "Comparing Source Code Branches"
date: 2009-09-16T01:54:00-07:00
excerpt: "During the more than three years I spent helping Agilent Technologies migrate their Internet site from their legacy, proprietary platform to Microsoft Office SharePoint Server (MOSS) 2007, we unfortunately never used Team Foundation Server (TFS). Instead..."
draft: true
categories: ["My System", "Development"]
tags: ["My System", "Core Development", "Visual Studio", "TFS", "Toolbox"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/09/16/comparing-source-code-branches.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/09/16/comparing-source-code-branches.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

During the more than three years I spent helping Agilent Technologies migrate  their Internet site from their legacy, proprietary platform to Microsoft Office  SharePoint Server (MOSS) 2007, we unfortunately never used Team Foundation Server  (TFS). Instead, we used Visual SourceSafe (VSS) in combination with a ["Work Items"
list in SharePoint](/blog/jjameson/2008/04/01/tfs-lite-for-wss-v2) that I've described in previous posts.

While I certainly prefer TFS over VSS, sometimes you simply have to concede that  you can't have everything you would like on a customer project and move on to actually  getting the work done.

However, just because we used VSS doesn't mean we didn't follow good Software  Configuration Management (SCM) principles. For example, as I've [described in the past](/blog/jjameson/2007/04/18/structure-visual-studio-solutions), I insist on using branching right from the start. Thus  when I setup the Visual Studio solution for the Agilent project, I created a **Main** branch and subsequently created branches for the various releases  (e.g. **v1.0**, **v2.0**, and **v3.0**).

The particular branch that a developer uses would thus depend on whether the  changes are for the next major release or a QFE (hotfix) for the version running  in Production. For example, after deploying the [Technical Support site](http://www.chem.agilent.com/en-US/Support) (i.e.  v2.0), we began working on the "General Site" (i.e. v3.0). [Note that in Agilent's  terminology, the "General Site" essentially refers to everything outside of Technical  Support, the Literature Library, and the Online Store (i.e. the "Buy" tab).]

However, since we didn't create the **v3.0** branch until shortly  before the v3.0 release, all v3 ("General Site") development was initially done  on the **Main** branch -- just like v2 (Tech Support) development was  initially done in **Main** prior to creating the **v2.0** branch.

This branching strategy works really well, regardless of which particular SCM  system you actually use. The key thing to remember is that all of the changes should  eventually make it into the **Main** branch (since that branch will  eventually be used to create another branch for the next major release).

The problem with VSS is that while it certainly supports branching, the merging  features, um...well, let's just say that they leave a lot to be desired ;-)

Personally speaking, I've never felt comfortable using the out-of-the-box merging  features in VSS, and instead always insisted on merging the changes from one branch  to another manually. TFS is obviously years ahead of VSS in terms of branching and  merging, but as I said before, sometimes you simply have to deal with what you've  got.

Fortunately, long before the Agilent project, I had previously created my own  process that takes a great deal of the "pain" out of manually merging source code.  Here is what I came up with.

In my [Toolbox](/blog/jjameson/2007/03/22/backedup-and-notbackedup),  I have two simple scripts: DiffBranches.cmd and CopyBranch.cmd.

Here are the contents of DiffBranches.cmd:

```
@echo off

setlocal

REM set DIFFTOOL=Windiff.exe
set DIFFTOOL=C:\NotBackedUp\Public\Toolbox\DiffMerge\DiffMerge.exe

set BRANCH1=%1
set BRANCH2=%2

if ("%BRANCH1%") == ("") set BRANCH1=Main

if ("%BRANCH2%") == ("") set BRANCH2=v3.0

call CopyBranch.cmd "%BRANCH1%" "%BRANCH1%_tmp"

call CopyBranch.cmd "%BRANCH2%" "%BRANCH2%_tmp"

"%DIFFTOOL%" "%BRANCH1%_tmp" "%BRANCH2%_tmp"
```

As you can see, there's not much to it. I simply make temporary copies of the  two branches (i.e. by copying the branch folder into a new folder appended with  "\_tmp") and then use my "Diff Tool" to compare the two folders. Originally, I used  WinDiff, but once I [discovered DiffMerge](/blog/jjameson/2009/03/24/diffmerge-a-better-differencing-tool), I quickly switched to using it exclusively for all of  my "diff'ing" activities.

The real "magic" lies in CopyBranch.cmd:

```
@echo off

setlocal

set BRANCH1=%1
set BRANCH2=%2

if ("%BRANCH1%") == ("") set BRANCH1=Main

if ("%BRANCH2%") == ("") set BRANCH2="%BRANCH1%_tmp"

robocopy "%BRANCH1%" "%BRANCH2%" /E /MIR /XD bin obj TestResults /XF *.scc *.suo *.user *.vspscc
```

Note that I put the word *magic* in quotes because it's really not magic  at all. When copying a source code branch, I simply use robocopy.exe and specify  the following options:

**robocoby.exe command-line options used in CopyBranch.cmd**

| Command-Line Option | Comments |
| --- | --- |
| /E | Copy subdirectories, including empty ones |
| /MIR | Mirror the directory tree, thus ensuring that when I subsequently run
CopyBranch.cmd, any deleted files from the original branch are removed from
the "temporary" copy of the branch) |
| /XD bin obj TestResults | Exclude directories matching the given names/paths, thus skipping the
compiled output (i.e. bin and obj) as well as the test results generated
by running unit tests from within Visual Studio |
| /XF \*.scc \*.suo \*.user \*.vspscc  | Exclude files matching the given names/paths/wildcards, thus skipping
source code control files (\*.scc and \*.vspscc), and Visual Studio solution/project
user-specific options (i.e. \*.suo and \*.user) |

For example, let's suppose that I've checked in some changes to the **v3.0** branch that need to be propagated to the **Main** branch.  I would open a command prompt and run the following:

{{< console-block-start >}}

C:\NotBackedUp\Agilent&gt;{{< kbd "DiffBranches.cmd v3.0 Main" >}}

{{< console-block-end >}}

After the two branches are copied to their respective temporary folders, DiffMerge.exe  is launched to compare the two branches. Since I exclude many of the directories  and files that are *expected* to differ between the two branches, I can quickly  view only the differences that I am interested in. I can even use DiffMerge.exe  to apply the changes interactively (checking out the files to be updated beforehand  as necessary). This greatly reduces the effort involved in manually merging changes  from one branch into another.

Note that it takes a little bit of time to copy a branch the first time, but  this is lightning fast on subsequent runs. Hence why I don't delete the "\_tmp" folders  after comparing two branches.

I occasionally use this process even on projects where we've used Team Foundation  Server. For example, I sometimes create a new workspace in TFS to represent a "temporary  branch" where I want to experiment with some substantial changes to the source code  (while I still have pending changes in my main workspace, that I may have shelved  but don't want to check-in just yet). Using a little robocopy.exe and DiffMerge.exe,  I can more easily compare two branches or workspaces.

