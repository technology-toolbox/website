---
title: "Building SharePoint 2010 Solutions on a TFS 2010 Build Server"
date: 2011-02-26T20:41:00-07:00
excerpt: "Last year I wrote a post about building Microsoft Office SharePoint Server (MOSS) 2007 solutions on a Team Foundation Server (TFS) 2010 build server , which talked about copying various SharePoint assemblies to a \"Reference Assemblies\" folder and adding..."
aliases: ["/blog/jjameson/archive/2011/02/26/building-sharepoint-2010-solutions-on-a-tfs-2010-build-server.aspx"]
draft: true
categories: ["Development", "SharePoint"]
tags: ["Visual Studio", "TFS", "SharePoint 
			2010"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/02/27/building-sharepoint-2010-solutions-on-a-tfs-2010-build-server.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/02/27/building-sharepoint-2010-solutions-on-a-tfs-2010-build-server.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that
> blog ever goes away.

Last year I wrote a post about
[building Microsoft Office SharePoint Server (MOSS) 2007 solutions on a Team
Foundation Server (TFS) 2010 build server](/blog/jjameson/2010/05/04/building-moss-2007-solutions-on-a-tfs-2010-build-server), which talked about copying various
SharePoint assemblies to a "Reference Assemblies" folder and adding a corresponding
registry key for MSBuild to locate the assemblies.

It's nice to see that a similar process has already been covered on MSDN
for SharePoint 2010:

{{< reference title="How to Build SharePoint Projects with TFS Team Build" linkHref="http://msdn.microsoft.com/en-us/library/ff622991.aspx" >}}

However, there are few things I noticed about this MSDN article.

First, the path for the registry key on 64-bit systems is incorrect (although
you should be able to easily figure out what the correct path is, once you are
navigating down through the registry). For the record:

`HKEY_LOCAL_SYSTEM\SOFTWARE\Microsoft\Wow6432Node\.NETFramework\v2.0.50727\AssemblyFoldersEx\SharePoint14]@="<AssemblyFolderLocation>"`

should be:

`HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\.NETFramework\v2.0.50727\AssemblyFoldersEx\SharePoint14]@="<AssemblyFolderLocation>"`

Second, the MSDN article instructs you to copy several Visual Studio SharePoint
assemblies to the GAC on the build server:

{{< blockquote "font-italic" >}}

**Copy the SharePoint Tool Assemblies and Files to the GAC
**The following assemblies must be copied to the GAC of the build
system:

- Microsoft.VisualStudio.SharePoint.Designers.Models.dll
- Microsoft.VisualStudio.SharePoint.Designers.Models.Features.dll
- Microsoft.VisualStudio.SharePoint.Designers.Models.Packages.dll
  Microsoft.VisualStudio.SharePoint.dll

{{< /blockquote >}}

Since I chose to install Visual Studio 2010 on my build server (DAZZLER),
then I shouldn't have to install *any* additional Visual Studio assemblies
on my build server. Everything should just work. When I looked at the GAC on
my SharePoint Server 2010 development VM (FOOBAR5), I didn't see these assemblies,
and since FOOBAR5 doesn't have any trouble building SharePoint projects without
these assemblies in the GAC, then I didn't expect DAZZLER would have any problems
either.

Lastly, as I mentioned in my post last year regarding building MOSS 2007
solutions with TFS 2010, you'll probably want to copy more assemblies than those
that are directly referenced in you project -- in order to avoid code analysis
warnings, like the following:

{{< blockquote "font-italic" >}}

CA0060 : The indirectly-referenced assembly 'Microsoft.SharePoint.Client.ServerRuntime, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c' could not be found. This assembly is not required for analysis, however, analysis results could be incomplete. This assembly was referenced by: C:\Program Files\Reference Assemblies\Microsoft\SharePoint v4\Microsoft.SharePoint.dll.

{{< /blockquote >}}

Here are the assemblies that I ended up copying from FOOBAR5 (which has SharePoint
Server 2010 installed) to DAZZLER (which does not have SharePoint Server 2010
installed):

**Reference Assemblies for Building SharePoint 2010 Projects**

| Assembly | Source Location on SharePoint 2010 Server |
| --- | --- |
| Microsoft.BusinessData.dll | ISAPI |
| Microsoft.HtmlTrans.Interface.dll | GAC\_MSIL |
| Microsoft.IdentityModel.dll | GAC\_MSIL |
| Microsoft.Office.Server.dll | ISAPI |
| Microsoft.Office.Server.UI.dll | GAC\_MSIL |
| Microsoft.Office.Server.UserProfiles.dll | ISAPI |
| Microsoft.SharePoint.AdministrationOperation.dll | GAC\_MSIL |
| Microsoft.SharePoint.Client.ServerRuntime.dll | GAC\_MSIL |
| Microsoft.SharePoint.Diagnostics.dll | GAC\_MSIL |
| Microsoft.SharePoint.dll | ISAPI |
| Microsoft.SharePoint.Dsp.dll | GAC\_MSIL |
| Microsoft.SharePoint.Library.dll | GAC\_MSIL |
| Microsoft.SharePoint.Powershell.dll | GAC\_MSIL |
| Microsoft.SharePoint.Publishing.dll | ISAPI |
| Microsoft.SharePoint.Search.dll | ISAPI |
| Microsoft.SharePoint.Security.dll | ISAPI |
| Microsoft.SharePoint.Taxonomy.dll | ISAPI |
| microsoft.sharepoint.WorkflowActions.dll | ISAPI |
| Microsoft.Web.Administration.dll | GAC\_MSIL |
| Microsoft.Web.CommandUI.dll | ISAPI |
| Microsoft.Web.Design.Server.dll | GAC\_MSIL |

In case it's not immediately obvious, "ISAPI" in the above table means:

> C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\14\ISAPI

...and "GAC\_MSIL" indicates the corresponding assembly folder under C:\Windows\assembly\GAC\_MSIL.
For example:

> C:\Windows\assembly\GAC\_MSIL\Microsoft.BusinessData\14.0.0.0\_\_71e9bce111e9429c

I may need to copy additional SharePoint assemblies to my build server in
the future, but for now, these are sufficient to compile my current solutions
without any warnings.

> **Update (2011-03-14)**
>
> I also recommend you remove extraneous SharePoint assemblies from your build output, as described in [one of my later posts](/blog/jjameson/2011/03/14/quot-build-bloat-quot-part-2-a-k-a-removing-extraneous-items-from-sharepoint-visual-studio-projects).

