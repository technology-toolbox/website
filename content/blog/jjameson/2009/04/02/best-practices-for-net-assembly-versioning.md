---
title: "Best Practices for .NET Assembly Versioning"
date: 2009-04-02T22:59:00+08:00
excerpt: "Whenever a new .NET assembly project is created in Visual Studio, a file named AssemblyInfo is created that contains attributes used to define the version of the assembly during compilation. 
 Using assembly versions effectively enables various team..."
draft: true
categories: ["My System", "Development"]
tags: ["My System", "Core Development", "Visual Studio"]
---

> **Note**
> 
>             This post originally appeared on my MSDN blog:  
>   
> 
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/04/03/best-practices-for-net-assembly-versioning.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/04/03/best-practices-for-net-assembly-versioning.aspx)
> 
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.


Whenever a new .NET assembly project is created in Visual Studio, a file named AssemblyInfo         is created that contains attributes used to define the version of the assembly during         compilation.

Using assembly versions effectively enables various team members to identify deployed         assemblies and helps troubleshoot problems that may occur in a particular environment         (e.g. Development, Test, or Production).

Assembly versions consist of four different parts ({Major Version}.{Minor Version}.{Build         Number}.{Revision}):
<dl>        <dt>Major Version</dt>
        <dd>
            Manually incremented for major releases, such as adding many new features to the
            solution.</dd>
        <dt>Minor Version</dt>
        <dd>
            Manually incremented for minor releases, such as introducing small changes to existing
            features.</dd>
        <dt>Build Number</dt>
        <dd>
            Typically incremented automatically as part of every build performed on the Build
            Server. This allows each build to be tracked and tested.</dd>
        <dt>Revision</dt>
        <dd>
            Incremented for QFEs (a.k.a. "hotfixes" or patches) to builds released into the
            Production environment (PROD). This is set to zero for the initial release of any
            major/minor version of the solution.</dd></dl>
As a general guideline, it is best to use the same version number for all assemblies         compiled as part of the solution. This is easily accomplished using [linked files in Visual Studio solutions](/blog/jjameson/2009/04/02/linked-files-in-visual-studio-solutions).

When building the solution, there are two version numbers that need to be considered:         the file version number and the .NET assembly version number.

###         File Version

The [AssemblyFileVersionAttribute](http://msdn.microsoft.com/en-us/library/system.reflection.assemblyfileversionattribute%28VS.71%29.aspx) should be incremented automatically as part         of the build process. At the beginning of the build process, before the build label         is applied in the source control system, the Build Number portion of the file version         is incremented (or the Revision, if building a QFE).

The file version should be isolated in its own file (e.g. AssemblyVersionInfo.cs)         to easily automate the process of incrementing it with every build. This file can         then be "linked into" each project in the solution, thus ensuring all of the various         assemblies in the solution share the same file version for a particular build.


> **Update (2010-04-22)**
> 
>             In a follow-up post, I provide details for how I recommend [incrementing the assembly version for each build](/blog/jjameson/2010/03/25/incrementing-the-assembly-version-for-each-build) (assuming you are using
>             Team Foundation Server).


###         Assembly Version

The [AssemblyVersionAttribute](http://msdn.microsoft.com/en-us/library/system.reflection.assemblyversionattribute.aspx) is the version that .NET uses when linking assemblies.         This number is not incremented with every build to avoid having to specify binding         redirects.

Assembly versions are incremented manually when branching the code for a release         (to the PROD environment).

The assembly version number should be specified in a "shared assembly info" file         (e.g. [SharedAssemblyInfo.cs](/blog/jjameson/2009/04/03/shared-assembly-info-in-visual-studio-projects)), thus ensuring that all of the various assemblies         in the solution have the same assembly version.

By default, the **Product version** shown in the file properties window         is the same as the value specified for **AssemblyFileVersionAttribute**.         Setting [AssemblyInformationalVersionAttribute](http://msdn.microsoft.com/en-us/library/system.reflection.assemblyinformationalversionattribute%28VS.71%29.aspx) to be the same as **AssemblyVersionAttribute**         ensures the **Product version** shown in the file properties window         matches the **Version **displayed in the [GAC shell extension](http://msdn.microsoft.com/en-us/library/34149zk3.aspx).

