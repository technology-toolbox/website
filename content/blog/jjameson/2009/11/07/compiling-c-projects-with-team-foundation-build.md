---
title: Compiling C++ Projects with Team Foundation Build
date: 2009-11-07T08:24:00-07:00
description:
  'As I mentioned in my previous post , this week I incorporated Password Minder
  into my "Toolbox" Visual Studio solution that is scheduled to build daily
  through Team Foundation Server (TFS). It''s not that I really need daily builds
  of Password Minder;...'
aliases:
  [
    "/blog/jjameson/archive/2009/11/06/compiling-c-projects-with-team-foundation-build.aspx",
    "/blog/jjameson/archive/2009/11/07/compiling-c-projects-with-team-foundation-build.aspx",
  ]
categories: ["Development"]
tags: ["Core Development", "TFS"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/11/07/compiling-c-projects-with-team-foundation-build.aspx"
---

As I mentioned in my
[previous post](/blog/jjameson/2009/11/07/using-password-minder-to-manage-your-passwords),
this week I incorporated Password Minder into my "Toolbox" Visual Studio
solution that is scheduled to build daily through Team Foundation Server (TFS).

It's not that I really need daily builds of Password Minder; rather it's just
been something on my "TO DO" list for a long time and I finally got around to
doing it. Unfortunately, it wasn't without issue.

In case you are not familiar with Password Minder, it is mostly written in C#,
but it includes one C++ project (for the NativeHelpers.dll).

Thus when I added the Password Minder projects to my "Toolbox" solution, I woke
up the next morning to a notification from Team Foundation Server that my build
failed. [No, I don't have some sort of alarm that goes off when one of my builds
break. By "notification" I am referring to the e-mail message that I found in my
inbox when I sat down at the computer.]

Clicking the build log referenced in the e-mail message, I quickly discovered
the following:

{{< log-excerpt >}}

<samp> Using "VCBuild" task from assembly "Microsoft.Build.Tasks.v3.5,
Version=3.5.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a". Task
"VCBuild"<br>Locating vcbuild.exe: not found at "c:\Program Files
(x86)\Microsoft Visual Studio
9.0\Common7\IDE\..\..\vc\vcpackages\vcbuild.exe".<br>Locating vcbuild.exe:
Visual C++ Express is not installed on this computer.<br>Locating vcbuild.exe:
falling back to the system PATH
variable.<br>C:\Users\svc-build\AppData\Local\Temp\Toolbox\Automated Build -
Main\Sources\Source\Toolbox.sln : error MSB3411: Could not load the Visual C++
component "VCBuild.exe". If the component is not installed, either 1) install
the Microsoft Windows SDK for Windows Server 2008 and .NET Framework 3.5, or 2)
install Microsoft Visual Studio 2008.</samp>

{{< /log-excerpt >}}

That certainly is one of the best error messages I've seen in a long time. It
told me exactly what I needed to do to fix the problem. Well, almost...

Note that DAZZLER (a VM in the
["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) that is
my dedicated build server) does not have the full installation of Visual Studio 2008. Rather it only has the Team Foundation Build install. In keeping with best
practices, I try to keep the build server as "clean" as possible. That means no
Visual Studio, no SharePoint, etc.

Following option #1 from the log file, I proceeded to install the Microsoft
Windows SDK for Windows Server 2008 and .NET Framework 3.5 -- but, of course, I
didn't bother to review the ReadMe file that comes with it (at least not at
first).

Consequently, I was greeted with the following error on the next build attempt:

{{< log-excerpt >}}

<samp> c:\Windows\Microsoft.NET\Framework\v3.5\Microsoft.Common.targets :
warning MSB3428: Could not load the Visual C++ component "VCProjectEngine.dll".
To fix this, 1) install the Microsoft Windows SDK for Windows Server 2008 and
.NET Framework 3.5, 2) install Microsoft Visual Studio 2008 or 3) add the
location of the component to the system path if it is installed elsewhere.
System error code: 126.<br>
c:\Windows\Microsoft.NET\Framework\v3.5\Microsoft.Common.targets :
warning MSB3425: Could not resolve VC project reference
"..\NativeHelpers\NativeHelpers.vcproj".</samp>

{{< /log-excerpt >}}

That's when I discovered the following from the release notes for the SDK:

{{< div-block "fst-italic" >}}

> #### 5.1.1 VCBuild fails to compile or upgrade projects
>
> In order for VCBuild to run properly, vcprojectengine.dll needs to be
> registered. If vcprojectengine.dll is not registered, VCBuild.exe will fail
> with errors such as:
>
> On compile: <samp>warning MSB3422: Failed to retrieve VC project information through the VC project engine object model. System error code: 127.</samp>
>
> On upgrade: <samp>Failed to upgrade project file 'foo.vcproj'. Please make sure the file exists and is not write-protected.</samp>
>
> To workaround this issue, vcprojectengine.dll must be manually registered.
> From a Windows SDK command line window (as administrator in Vista:
>
> On an X86 machine, run:
>
> ```Batch
> cd %mssdk%\VC\bin
> regsvr32 vcprojectengine.dll
> ```
>
> On an X64 machine, run:
>
> ```Batch
> cd %mssdk%\VC\bin\X64
> regsvr32 vcprojectengine.dll
> ```

{{< /div-block >}}

Unfortunately, these instructions aren't quite right -- or at least they didn't
work verbatim in my environment. The workaround stated above makes you think
there's an environment variable (`%mssdk%`) that refers to the path where the
SDK is installed. However, this wasn't configured on DAZZLER.

Also, I didn't find my copy of VCProjectEngine.dll in an **x64** folder, but
rather in an **amd64** folder:

> C:\Program Files (x86)\Microsoft Visual Studio
> 9.0\VC\bin\amd64\VCProjectEngine.dll

I know most of you out there are not doing much (if any) C++ work anymore, but I
thought I should share this just in case you need it at some point.
