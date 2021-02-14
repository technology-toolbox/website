---
title: "\"Web-Enabled\" Projects and F5 Debugging with SharePoint"
date: 2009-09-29T03:07:00+08:00
excerpt: "In yesterday's post , I provided a sample walkthrough of the \"DR.DADA\" approach to developing solutions for Microsoft Office SharePoint Server (MOSS) 2007. However, I intentionally left out a few things because a) that post was already getting ridiculously..."
draft: true
categories: ["My System", "SharePoint", "Development"]
tags: ["My System", "MOSS 2007", "WSS v3", "Debugging"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/09/29/web-enabled-projects-and-f5-debugging-with-sharepoint.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/09/29/web-enabled-projects-and-f5-debugging-with-sharepoint.aspx)
> 
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

In [yesterday's post](/blog/jjameson/2009/09/28/sample-walkthrough-of-the-dr-dada-approach-to-sharepoint), I provided a sample walkthrough of the "DR.DADA" approach  to developing solutions for Microsoft Office SharePoint Server (MOSS) 2007. However,  I intentionally left out a few things because a) that post was already getting ridiculously  long, and b) I felt these were important enough to cover separately.

One of the incorrect statements I've heard a few times over the last couple of  years is that you can't do "<kbd>F5</kbd> debugging" when working with SharePoint.  Well, I suppose that in the strictest sense, this is a true statement -- assuming  you don't go crazy with post-build events (for example, to deploy your updated WSP,  re-GAC your assemblies, and recycle the application pool). Instead, most developers  -- including myself back in the early days of MOSS 2007 -- start debugging by attaching  to the IIS worker process (i.e. w3wp.exe).

However, when you have multiple instances of w3wp.exe (for example you are running  a couple of SharePoint Web applications in addition to Central Administration) it  can be tedious attaching to the right worker process. [In other words, the old keystroke  combination many of us grew accustomed to back in the days of working on a single  ASP.NET Web appliction -- specifically, pressing <kbd>CTRL+SHIFT+P</kbd> (to bring  up the **Attach To Process** dialog box), pressing <kbd>W</kbd> (to  scroll the list of processes down to w3wp.exe), followed by two quick presses of  the <kbd>Enter</kbd> key -- doesn't work anymore because we might attach to the  wrong worker process. Even worse, we might not be able to quickly tell which w3wp.exe  instance to attach to without expanding the **User Name** column --  or even worse still, having to use <samp>iisapp.vbs</samp> (in Windows Server 2003)  or <kbd>C:\Windows\System32\inetsrv\appcmd.exe list apppool</kbd> (in Windows  Server 2008) to determine which process to attach to.]

Don't fret...attaching to the right worker process to debug your SharePoint code *can* be very easy.

Suppose you've created a **Class Library** project in Visual Studio  for your feature, which includes things like a master page with code-behind, custom  Web Parts, event receivers, or perhaps even a feature receiver or two -- and now  you actually need to debug that code. [Note that there's actually a much [easier way to debug your feature receivers](/blog/jjameson/2007/03/22/what-s-in-a-name-defaultfeaturereceiver-vs-featureconfigurator), but for the purposes of this post,  suppose you actually want to debug activating a feature through **Site Settings**.]

First, you need to "Web-enable" your class library project. [I'm not sure if  "Web-enabling" is actually the official name for this -- in fact, I doubt it. However,  that's what I've been calling it for a few years now and it seems to describe the  concept to most people I tell this to.]

Unfortunately, there's no user interface in Visual Studio 2008 for "Web-enabling"  your project so you have to modify the MSBuild project file directly.

To Web-enable your C# class library project and configure for ASP.NET debugging:

1. In the **Solution Explorer** window, select the class library
   project.
2. Right-click the project name and then click **Unload Project**.
3. Right-click the unloaded project and then click **Edit {project name}**.
4. Below the `<ProjectGuid>`
   element, add the following:

`<ProjectTypeGuids>{349c5851-65df-11da-9384-00065b846f21};{fae04ec0-301f-11d3-bf4b-00c04f79efbc}</ProjectTypeGuids>`
5. Below the `<Import Project="$(MSBuildToolsPath)\Microsoft.CSharp.targets"  />` element, add the following:

`<Import Project="$(MSBuildExtensionsPath)\Microsoft\VisualStudio\v9.0\WebApplications\Microsoft.WebApplication.targets" Condition=""  />`
6. On the **File** menu, click **Close**. When prompted
to save the file, click **Yes**.
7. In the **Solution Explorer** window, right-click the project,
and then click **Reload Project**.
8. Wait for the project to finish loading, right-click the project name again,
and click **Properties**. Notice that there is now a **Web** tab in the project settings.
9. On the project settings **Web** tab, under the **Servers**section:
1. Clear the **Apply server settings to all users (store in project
file)** checkbox (since various members of the Development team might
use different URLs for their local SharePoint sites).
2. Select the **Use Custom Web Server** option, and in the
**Server Url** box, type the URL of your SharePoint site (e.g.
[http://fabweb-local](http://fabweb-local/)).
10. Close the project settings window.
11. In the **Solution Explorer** window, right-click the project,
and then click **Set as StartUp Project**.

> **Note**
> 
> In addition to seeing the **Web** tab in project settings,
> you will also find that adding items like ASPX pages and ASCX controls is
> much easier after "Web-enabing" your project.

To ensure ASP.NET debugging is enabled on the Web site [note these instructions  are for Windows Server 2008]:

1. Click **Start**, click **Administrative Tools**,
   and then click **Internet Information Services (IIS) Manager**.
2. In the **Connections** pane, expand the computer name.
3. Expand **Sites**.
4. Select the Web site or application that you want to debug.
5. Under the **ASP.NET** section, double-click **.NET Compilation**.
6. Under the **Behavior** section, ensure the value of **
   Debug** is set to **True**.
7. If necessary, click **Apply** in the **Actions** pane.

Assuming you have deployed your solution and activated your features, your can  now set a breakpoint and press <kbd>F5</kbd> to start debugging. Woohoo!

Now let's suppose that you find a bug in your code and need to make a change  -- but only to the code. In other words, you haven't modified any of your files  deployed to %ProgramFiles%\Common Files\Microsoft Shared\Web Server Extensions\12.

As I mentioned yesterday, all you need to do is press <kbd>CTRL+SHIFT+B</kbd>  to build your solution, GAC your updated assemblies, and recycle the application  pool:

C:\NotBackedUp\Fabrikam\Demo\Main\Source\Publishing\DeploymentFiles\Scripts&gt;<kbd>"GAC Assemblies.cmd"</kbd>

```
Installing assembly: Fabrikam.Demo.CoreServices.dll (Debug)
Assembly successfully added to the cache
Installing assembly: Fabrikam.Demo.Publishing.dll (Debug)
Assembly successfully added to the cache
Done
```

C:\NotBackedUp\Fabrikam\Demo\Main\Source\Publishing\DeploymentFiles\Scripts&gt;<kbd>C:\Windows\System32\inetsrv\appcmd.exe recycle apppool "SharePoint - foobar-local80"</kbd>

```
"SharePoint - foobar-local80" successfully recycled
```

You can then simply press <kbd>F5</kbd> to start debugging again. Woohoo, indeed!

I hope this makes you a happier and more productive SharePoint developer.

