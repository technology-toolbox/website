---
title: "\"Development Cheat Sheets\""
date: 2013-05-01T01:00:45-06:00
excerpt:
  "In this post, I explain how I like to use Microsoft OneNote to manage what I
  call \"development cheat sheets\" -- which are especially useful when
  developing SharePoint solutions."
aliases:
  [
    "/blog/jjameson/archive/2013/04/30/development-cheat-sheets.aspx",
    "/blog/jjameson/archive/2013/05/01/development-cheat-sheets.aspx",
  ]
draft: true
categories: ["Development", "My System", "SharePoint"]
tags: ["My System", "SharePoint 2010"]
---

I like to use Microsoft OneNote to manage a number of what I call "development
cheat sheets." These come in very handy when doing SharePoint development,
because -- at least in my experience -- this is rarely as simple as pressing {{<
kbd "F5" >}} after making some changes in Visual Studio. I know that's what the
SharePoint/Visual Studio guys would like it to be, but I personally find it
faster (and more reliable) to use my "DRDADA" PowerShell deployment scripts.

The following figure shows an example cheat sheet for the Main branch of my
Fabrikam Demo solution.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/My-System/Development-cheat-sheet-Fabrikam-Demo-Main-branch-600x393.png"
alt="Development cheat sheet - Fabrikam Demo - Main branch" class="screenshot"
height="393" width="600"
title="Figure 1: Development \"cheat sheet\" - Fabrikam Demo - Main branch" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/My-System/Development-cheat-sheet-Fabrikam-Demo-Main-branch-1098x719.png)

Essentially all I need to do after starting one of my SharePoint development VMs
is:

1. Open Visual Studio and “Get Latest” on the solution
2. Package the "CoreServices.SharePoint" and “Web” projects (in other words,
   build the WSPs for the two SharePoint projects in this particular solution)
3. Start an instance of PowerShell then copy/paste the first four lines from my OneNote page:

   ```PowerShell
   Add-PSSnapin Microsoft.SharePoint.PowerShell
   cd "C:\NotBackedUp\Fabrikam\Demo\Main\Source\Deployment Files\Scripts"
   cls
   & '.\Redeploy Features.ps1'
   ```

Note that the “Redeploy Features.ps1” script is essentially the same actions
that are performed when you right-click the "CoreServices.SharePoint" and “Web”
projects in Visual Studio and click **Deploy**.

When I have only made minor changes to one or more files (either code or an
existing ASPX or ASCX file) then I typically just run the “Upgrade
Solutions.ps1” script, since this shaves precious time off the iterative
development process:

```PowerShell
cls
& '.\Upgrade Solutions.ps1'
```

If, for whatever reason, I want to run the individual scripts invoked by the
“Redeploy Features.ps1” script, then I have a series of commands in my OneNote
page to “DRDADA” the solutions/features:

```PowerShell
cls
& '.\Deactivate Features.ps1'
& '.\Retract Solutions.ps1'
& '.\Delete Solutions.ps1'
& '.\Add Solutions.ps1'
& '.\Deploy Solutions.ps1'
& '.\Activate Features.ps1'
```

Note that when you run the “Create Site Collections.ps1” script in the Fabrikam
solution, it creates the root site collection in the Web application (if it
doesn't already exist). Therefore if I've made some changes to the site
collection that I quickly want to discard, I can run the Remove-SPSite cmdlet,
followed by “Create Site Collections.ps1” and “Redeploy Features.ps1” in order
to quickly get back to a “known good” state:

```PowerShell
cls
Remove-SPSite http://fabrikam-local/ -Confirm:$false
& '.\Create Site Collections.ps1'
& '.\Redeploy Features.ps1'
```

{{< div-block "note" >}}

> **Note**
>
> The “Delete Site Collections.ps1” script will delete all of the site
> collections created by the “Create Site Collections.ps1” script, so I
> typically just use the Remove-SPSite cmdlet directly (as illustrated above).

{{< /div-block >}}

Lastly, if I need to build (or rebuild) my environment and just want the
absolute least amount of human effort (by sacrificing a little more “wait”
time), I can run the “Rebuild Web Application.ps1” script and then run the "Test
Console" utility to populate some sample content. In my SharePoint development
VMs, this process typically takes about two minutes:

```PowerShell
cls
& '.\Rebuild Web Application.ps1' -Confirm:$false
pushd ..\..\Tools\TestConsole\bin\Debug
.\Fabrikam.Demo.Tools.TestConsole.exe
```

I should also mention how I create different pages in OneNote for each branch in
TFS that I frequently work on. For example, the following figure shows the cheat
sheet for the **SharePointExtranet** branch that I created for the code sample
included in
[yesterday's post](/blog/jjameson/2013/04/30/installation-guide-for-sharepoint-server-2010-and-office-web-apps).

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/My-System/Development-cheat-sheet-Fabrikam-Demo-SharePointExtranet-branch-600x524.png"
alt="Development cheat sheet - Fabrikam Demo - SharePointExtranet branch"
class="screenshot" height="524" width="600"
title="Figure 2: Development \"cheat sheet\" - Fabrikam Demo - SharePointExtranet branch" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/My-System/Development-cheat-sheet-Fabrikam-Demo-SharePointExtranet-branch-1190x1040.png)

The commands at the beginning are similar to the ones for the **Main** branch
(with the exception of the path for the TFS workspace and the URL of the Web
application). However, this page also contains some information about merging.
For example, I frequently need to merge changes from the **Main** branch to the
**SharePointExtranet** branch (i.e. forward integration) or from the
**SharePointExtranet** branch to the **Main** branch (i.e. reverse integration).

While I typically perform these merges using the Source Control Explorer in
Visual Studio, I use OneNote to keep a copy of the check-in comments I
frequently use when merging. Also note that when I need to discard changes from
one branch to the other, I need to use tf.exe from the command line (I'm not
aware of any way to discard the changes when merging in Source Control
Explorer).

Finally, I want to point out that while the cheat sheets often start out rather
simple, they typically evolve over time to include more and more shortcuts. For
example, here is the OneNote page for the Main branch of the Electronic Lab
Notebook (ELN) project that I worked on at Dow Chemical. You can see there's
quite a bit more in this one than the cheat sheets for the Fabrikam Demo site.
For example, the ELN solution includes custom SharePoint workflows and therefore
it is necessary to restart the SharePoint Timer service after redeploying (or
upgrading) the ELN solutions (WSPs).

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/My-System/Development-cheat-sheet-Dow-ELN-Main-branch-600x461.png"
alt="Development cheat sheet - Dow ELN - Main branch" class="screenshot"
height="461" width="600"
title="Figure 3: Development \"cheat sheet\" - Dow ELN - Main branch" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/My-System/Development-cheat-sheet-Dow-ELN-Main-branch-1328x1021.png)
