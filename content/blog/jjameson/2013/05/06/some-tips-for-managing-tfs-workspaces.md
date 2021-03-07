---
title: "Some Tips for Managing TFS Workspaces"
date: 2013-05-06T12:51:05-06:00
excerpt: "Are you familiar with the \"tfpt scorch\" command? Have you ever used PowerShell to quickly cloak/uncloak folders in your TFS workspaces? If not, here are a couple of tips that could save you a little time."
aliases: ["/blog/jjameson/archive/2013/05/06/some-tips-for-managing-tfs-workspaces.aspx"]
draft: true
categories: ["Development"]
tags: ["My System", "PowerShell", "TFS"]
---

I admit it...I'm sort of a neat freak. Call it a "touch of OCD" or whatever term
you like, but sometimes I really like to "reset" whatever I'm working on and
start over "fresh."

On the projects I typically work on, compiling the entire solution takes no more
than a couple of minutes in my development environment. (The i7 2600K, 32 GB of
RAM, and SSD in my desktop help immensely in this area.) Therefore, I don't mind
periodically rebuilding the entire solution, as opposed to always performing an
incremental build.

I also lean heavily on Team Foundation Server (TFS) when I'm doing what I call
"experimental development." By that I mean, checking out files and moving code
blocks -- or even entire files -- around, making "what if" changes to existing
code, and subsequently "undoing" my changes once the "experiment" is complete
(potentially shelving my changes first for later use).

Often I want to ensure that my TFS workspace matches the exact state of the
solution in source control. I used to do this the "hard way" by:

1. Cloaking the solution folder in Source Control Explorer
2. Using Windows Explorer, deleting the solution folder on disk
3. Uncloaking the solution folder in Source Control Explorer

Sometime last year, I discovered there's an easier way to do this, assuming you
have installed the TFS Power Tools -- which I certainly hope you have (if, for
no other reason, than to
[leverage additional check-in policies](/blog/jjameson/2009/10/31/recommended-check-in-policies-for-team-foundation-server)).

From a Visual Studio command prompt, simply use the {{< kbd "tfpt scorch" >}}
command. For example:

```
cd "\NotBackedUp\Dow\Collaboration\ELN HD"
tfpt scorch Main /r
```

Note that {{< kbd "/r" >}} is short for {{< kbd "/recursive" >}}.

You'll be prompted to confirm the changes to the workspace (assuming it doesn't
exactly match the items in source control).

This works great for cleaning up a single TFS workspace.

However, what about when I have just finished building a new development VM, or
I want to "reset" an existing VM? Depending on the complexity of the source code
structure in TFS (e.g. Dev branches, Release branches, extraneous folders I
don't want to sync, etc.), it can take a while to click through the context menu
in Source Control Explorer to cloak/uncloak various folders.

In that case, you can combine the {{< kbd "tf.exe" >}} utility with PowerShell
to make this a breeze. For example, suppose I've been working on various Dev or
Release branches in the Dow Collaboration team project and now I want to clean
up my local workspace (in other words, I only need the Main branch).

I start by invoking PowerShell from a Visual Studio command prompt, changing to
the root folder for the project, and then storing the output from the {{< kbd
"tf dir" >}} command in a variable:

```
PowerShell
cd C:\NotBackedUp\Dow\Collaboration
$output = tf dir
```

At this point, the `$output` variable contains something like:

```
$/Dow Collaboration:
$2007
$2010
$BuildProcessTemplates
$Byron's Code
$CoatingsSharePointFeature
$CoatingsSharePointFeature(before resource export)
$CoatingsSiteDirectory-v1
$CoatingsSiteDirectory-v2
$CoreServices
$ELN HD
...
$ResearchPortal
$SharePointCustomBuildWorkflow

21 item(s)
```

From this, I can parse the folder names in the team project. Note that I don't
want the first line or the last few lines. No problem...

```
$tfFolders = $output[1..($output.Length - 3)]
```

Now the `$tfFolders` variable contains something like:

```
$2007
$2010
$BuildProcessTemplates
$Byron's Code
$CoatingsSharePointFeature
$CoatingsSharePointFeature(before resource export)
$CoatingsSiteDirectory-v1
$CoatingsSiteDirectory-v2
$CoreServices
$ELN HD
...
$ResearchPortal
$SharePointCustomBuildWorkflow
```

With this, I can quickly run as command to cloak all of the folders. However,
notice the dollar signs at the beginning of each line. I'll need to trim those
off when passing each folder to the {{< kbd "tf workfold /cloak" >}} command:

{{< console-block-start >}}

$tfFolders | foreach { tf workfold /cloak $\_.Substring(1) }

{{< console-block-end >}}

Now suppose that I want to build the **CoreServices** project. Consequently I
need to uncloak that folder and get the latest version from TFS. However, in
this particular case, the **CoreServices** folder contains a number of branches
(e.g. multiple "lab" development branches under the **Dev** folder, the **Main**
branch, and multiple release branches under the **Release** folder).

Here are the commands to only get the **Main** branch:

```
tf workfold /decloak CoreServices
tf workfold /cloak CoreServices/Dev
tf workfold /cloak CoreServices/Release
tf get CoreServices /recursive
```

At this point, my workspace contains an exact copy of the **Main** branch -- and
only the **Main** branch -- for the **CoreServices** project.

Since I probably also want to build the latest version of the ELN and Research
Portal solutions, I can use similar commands for those folders:

```
tf workfold /decloak "ELN HD"
tf workfold /cloak "ELN HD/Business Data Connectivity Models"
tf workfold /cloak "ELN HD/Dev"
tf workfold /cloak "ELN HD/POC Code"
tf workfold /cloak "ELN HD/Release"
tf workfold /cloak "ELN HD/Security"
tf workfold /cloak "ELN HD/Storyboarding"
tf workfold /cloak "ELN HD/UserInterface"
tf get "ELN HD" /recursive
tf workfold /decloak "ResearchPortal"
tf workfold /cloak "ResearchPortal/Dev"
tf workfold /cloak "ResearchPortal/Release"
tf get "ResearchPortal" /recursive
```

Note that the **ELN HD** folder contains a number of "prototype" folders that
probably should have been moved under the Dev folder by now...but you get the
point.

I keep a copy of these "scripts" in OneNote so I can easily run them again
whenever I want.

