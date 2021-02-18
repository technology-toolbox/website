---
title: "Suppress CS1607 Warning for TFS 2010 Builds"
date: 2010-05-05T00:32:00-07:00
excerpt: "Here's another issue I encountered when upgrading my Team Foundation Server (TFS) 2008 environment to TFS 2010 ... 
 While it's generally a good assumption that a solution that builds without error in Visual Studio 2008 (and on a TFS 2008 build server..."
aliases: ["/blog/jjameson/archive/2010/05/05/suppress-cs1607-warning-for-tfs-2010-builds.aspx"]
draft: true
categories: ["Development"]
tags: ["Core Development", "Visual Studio", "TFS"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/05/05/suppress-cs1607-warning-for-tfs-2010-builds.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/05/05/suppress-cs1607-warning-for-tfs-2010-builds.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

Here's another issue I encountered when [upgrading my Team Foundation Server (TFS) 2008 environment to TFS 2010](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010-overview)...

While it's generally a good assumption that a solution that builds without error in Visual Studio 2008 (and on a TFS 2008 build server) will build without error after updating the solution to Visual Studio 2010, this isn't always the case. For example, I mentioned before an error that I encountered due to the fact that [test projects in Visual Studio 2010 must target .NET Framework 4](/blog/jjameson/2010/04/28/test-projects-in-visual-studio-2010-must-target-net-framework-4).

I also found that one of my solutions compiled without error in Visual Studio 2010 on my local development machine, but failed with the following errors when performing a "Team Build" (i.e. on a TFS build agent):

- error CS1607: Assembly generation -- Referenced assembly 'mscorlib.dll' targets a different processor
- error CS1607: Assembly generation -- Referenced assembly 'System.Data.dll' targets a different processor
- CSC: Assembly signing failed; output may not be signed -- Error signing assembly -- The system cannot find the file specified

Note that [CS1607](http://msdn.microsoft.com/en-us/library/4a0640cd.aspx) is actually just a compiler warning -- meaning that it doesn't necessarily break the build. However if, like me, you configure your projects to treat all warnings as errors, then it will indeed break the build.

I should also point out that this error occurred on my [ConvertToDataSet](/blog/jjameson/2009/10/08/importing-pages-into-moss-2007-from-an-excel-file) utility project -- which I had previously configured to target the x86 platform (an unfortunate necessity in order to read Excel files via the 32-bit OLEDB drivers). The strange part is that this project configuration compiled without issue on my old TFS 2008 build server (as well as in Visual Studio 2010 on my local development machine).

At first, I suspected that it might be caused by the fact that the project settings were configured as follows:

- Platform: Any CPU
- Platform target: x86

Consequently, I created a new platform (x86) and changed the solution to compile **Mixed Platforms** (instead of **Any CPU**). Unfortunately, that didn't help.

I even tried changing one of the "problem" assemblies to specify a fully-qualified assembly name (including the processor architecture), like this...

```
<Reference Include="System.Data, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089, ProcessorArchitecture=x86" />
```

...but that didn't work either.

I ended resolving the issue by keeping **Treat warnings as errors** set to **All**, but adding the following to the project file:

```
<WarningsNotAsErrors>1607</WarningsNotAsErrors>
```

I pasted this immediately below the following elements (for the Debug|x86 and Release|x86 configurations):

```
<TreatWarningsAsErrors>true</TreatWarningsAsErrors>
```

Note that Visual Studio complains a little that `<WarningsNotAsErrors>` is not an expected element according the schema, but I can assure this is, in fact, a [valid option for the C# compiler](http://msdn.microsoft.com/en-us/library/microsoft.build.tasks.csc.warningsnotaserrors.aspx), and the project subsequently builds just fine.

