---
title: "Updated Thoughts on WSPBuilder"
date: 2009-03-31T01:55:00+08:00
excerpt: "Several weeks ago, I wrote a post titled \" Why I'm Not a Fan of WSPBuilder .\" Shortly thereafter, I received a message from Carsten Keutmann, the creator of WSPBuilder. 
 Here is the \"almost\" unabridged version of the email exchange (headers and signatures..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
> 
>       This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/31/updated-thoughts-on-wspbuilder.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/31/updated-thoughts-on-wspbuilder.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

Several weeks ago, I wrote a post titled "[Why
I'm Not a Fan of WSPBuilder](/blog/jjameson/2009/03/06/why-i-m-not-a-fan-of-wspbuilder)." Shortly thereafter, I received a message from Carsten Keutmann, the creator of WSPBuilder.

Here is the "almost" unabridged version of the email exchange (headers and signatures removed):

1. <cite>Keutmann</cite>

> I would like to know that you are not talking about the STSDEV project, but it is really the WSPBuilder project. Because the WSPBuilder do not build the WSP package on the <kbd>CTRL+SHIFT+B</kbd> command, however the STSDEV does.
> 
> Please check up on this and let me know, furthermore I'll be happy to look at the WSPBuilder code for optimizations if necessary.
> 2. <cite>Jeremy</cite>

> It's definitely WSPBuilder. The solution contains folders for WSPBuilder\_x64 and WSPBuilder\_x86 (which contain executables, a config file, and a couple of other assemblies).
> 
> The WSP packages are built whenever I build the solution (<kbd>CTRL+SHIFT+B</kbd>) via MSBuild project files (including a common SharePoint.WSPBuilder.targets). Is this not the typical way people use WSPBuilder?
> 3. <cite>Keutmann</cite>

> The standard installation of WSPBuilder Extensions for Visual Studio do not build the WSP package when building the project (<kbd>CTRL-SHIFT-B</kbd>). Normally you have to build the WSP package by using the menu "Tools -&gt; WSPBuilder -&gt; Build WSP" command. (If you do not have this menu, then you do not have the Extensions installed.)
> 
> I only build the WSP package when I need to install/upgrade the WSP in my SharePoint farm. Otherwise I just rebuild the code and use the "Copy to GAC" command. It dosent make sense that you would build the WSP package every time that you rebuild your code.
> 
> So just remove the WSPBuilder.exe from the MSBuild script on you project and call the build of the WSP package manually when needed.
> 4. <cite>Jeremy</cite>

> Hmmm...I didn't install the WSPBuilder Extensions for Visual Studio in order to build the solution -- the team apparently already pulled in the necessary dependencies in order to work on a "vanilla" Visual Studio installation.
> 
> Does the SharePoint.WSPBuilder.targets file ship as part of your utility? I'm starting to wonder if this was something custom developed for this project...
> 5. <cite>Keutmann</cite>

> The SharePoint.WSPBuilder.targets file is not something that is shipped with WSPBuilder, so it must be something that someone in your project have added.
> 
> Therefore I would recommend that you remove the SharePoint.WSPBuilder.targets file from the developing machines and install the WSPBuilder Extensions for Visual Studio or use small bat scripts to run WSPBuilder manually when needed.
> 6. <cite>Jeremy</cite>

> Ah...well, it that case, it sounds like we **\*should\*** be treating WspBuilder the same as makecab.exe and calling it whenever an item in the project has been updated -- instead of whatever "magic" was put into the custom SharePoint.WSPBuilder.targets file.
> 
> In order to support automated builds (using the same configurations as the Development team uses), I want to avoid relying on add-ins and just stick with what comes out-of-the-box with MSBuild.
> 
> I'll take a look at this over the next couple of days and post an update to my blog. If the solution build times are comparable using makecab.exe and WSPBuilder.exe, then this would definitely change my perspective. Like I said before, I love the concept -- I just don't like the way it is performing on this particular project (which may very well be the result of how the team implemented it into the build process).
> 
> Thanks for pointing this out.

So, first of all, thanks to Keutmann (as he apparently prefers to be called) for contacting me about the issue I blogged about; and second, my apologies for taking this long to post an update!

It turns out that the very long build times I noted in my earlier post are really not due to WSPBuilder, but rather how WSPBuilder is integrated with this particular solution (à la the custom SharePoint.WSPBuilder.targets file).

To verify this, I had planned on performing an experiment in which I would eliminate the custom SharePoint.WSPBuilder.targets file and replace the four separate MSBuild projects in our solution (used to build the corresponding WSPs) with simple `<Exec>` tasks that invoke WspBuilder.exe to create the manifest files and corresponding WSPs.

In other words, I wanted to check the incremental build times when invoking WspBuilder.exe in the same [manner that I used to invoke makecab.exe](/blog/jjameson/2008/04/10/a-better-way-to-build-sharepoint-solution-packages-and-cab-files) on my previous project.

Unfortunately, after digging into this for a half hour, I realized this was going to take considerable time and effort. While it sounds like a relatively simple exercise, the reality is that the way our current solution is structured, one WSP actually packages multiple features. In other words, part of the "magic" performed by the custom SharePoint.WSPBuilder.targets file is to perform a "deep copy" of, for example, the "12" folders in multiple feature projects into a temporary folder. WSPBuilder is subsequently invoked in that temporary folder to create a combined manifest and package items into the WSP.

Conceptually, however, I do believe that if you choose to use WSPBuilder, then you should integrate it into your solution as a separate task in your project file (which simply uses an `<Exec>` task to invoke WspBuilder.exe with the various command-line options necessary to create your manifest and build the WSP).

Refer to my original post for more details on how to do this: [A Better Way to Build SharePoint Solution Packages (and CAB Files)](/blog/jjameson/2008/04/10/a-better-way-to-build-sharepoint-solution-packages-and-cab-files). Just substitute the use of makecab.exe with WspBuilder.exe and make the corresponding changes as necessary.

While Keutmann may think "it doesn't make sense that you would build the WSP package every time that you rebuild your code" and instead prefer to use the WSPBuilder Extensions for Visual Studio, personally I like to keep things simple and therefore I would much rather have <kbd>CTRL+SHIFT+B</kbd> build everything that needs to be built -- acknowledging that, like Keutmann, I bypass deploying the updated solution (and instead just GAC the updated assemblies) if I know that all that has changed is code and not, for example, master pages or page layouts (i.e. ASPX files).

Perhaps I'm just an old dog... ;-)

In my next post, I'll share my approach (a.k.a. the "DR.DADA" approach) for setting up a Visual Studio solution for SharePoint development.

