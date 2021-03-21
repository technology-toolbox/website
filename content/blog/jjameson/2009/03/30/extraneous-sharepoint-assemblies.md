---
title: Extraneous SharePoint Assemblies
date: 2009-03-30T08:10:00-06:00
excerpt:
  If you develop solutions for Microsoft Office SharePoint Server (MOSS) 2007,
  you may notice that certain SharePoint assemblies always get copied to your
  Visual Studio project output folder even though these referenced assemblies
  are configured with Copy...
aliases:
  [
    "/blog/jjameson/archive/2009/03/29/extraneous-sharepoint-assemblies.aspx",
    "/blog/jjameson/archive/2009/03/30/extraneous-sharepoint-assemblies.aspx",
  ]
draft: true
categories: ["SharePoint", "My System"]
tags: ["MOSS 2007", "Toolbox"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/03/30/extraneous-sharepoint-assemblies.aspx"
---

If you develop solutions for Microsoft Office SharePoint Server (MOSS) 2007, you
may notice that certain SharePoint assemblies always get copied to your Visual
Studio project output folder even though these referenced assemblies are
configured with **Copy Local** = **False**. Since these assemblies should always
refer to the ones installed by MOSS 2007 on the destination environment, you
obviously don't want to deploy these as part of your solution.

The "extraneous" files that I've seen copied are:

- Microsoft.Office.Server.Search.dll
- Microsoft.Office.Workflow.Feature.dll
- Microsoft.SharePoint.Portal.SingleSignon.dll
- Microsoft.SharePoint.Search.dll
- Microsoft.SharePoint.Search.xml

While I can't provide an explanation for why the **Copy Local** property is
ignored for these assemblies, I can provide you with a script to recursively
remove these files from your Visual Studio project structure.

Sometime last year, I created the following script and dropped it in my
SharePoint [Toolbox](/blog/jjameson/2007/03/22/backedup-and-notbackedup) folder
(\NotBackedUp\Public\Toolbox\SharePoint\Scripts\DeleteExtraneousSharePointAssemblies.vbs):

```VBA
Option Explicit

Dim strFolder
strFolder = "."

If (WScript.Arguments.Count = 1) Then
    strFolder = WScript.Arguments(0)
ElseIf (WScript.Arguments.Count > 1) Then
    WScript.Echo("Usage: DeleteExtraneousSharePointAssemblies.vbs [folder]")
    WScript.Quit(1)
End If

Dim fso
Set fso = CreateObject("Scripting.FileSystemObject")

Dim folder
Set folder = fso.GetFolder(strFolder)

If (InStr(folder, "Program Files") > 0) Then
    WScript.Echo("Error: Cannot delete assemblies from Program Files")
    WScript.Quit(1)
End If

Wscript.Echo "Scanning: " & folder.Path

DeleteExtraneousSharePointAssemblies folder

Sub DeleteExtraneousSharePointAssemblies(folder)
    Wscript.Echo "Scanning: " & folder.Path

    Dim file
    For Each file in folder.Files
        If file.Name = "Microsoft.Office.Server.Search.dll" _
            Or file.Name = "Microsoft.Office.Workflow.Feature.dll" _
            Or file.Name = "Microsoft.SharePoint.Portal.SingleSignon.dll" _
            Or file.Name = "Microsoft.SharePoint.Search.dll" _
            Or file.Name = "Microsoft.SharePoint.Search.xml" Then

            If (Not (file.Attributes And 1 = file.Attributes)) Then
                Wscript.Echo "Removing read-only flag from file: " & file.Path
                file.Attributes = file.Attributes XOr 1
            End If

            Wscript.Echo "Deleting " & file.Path
            file.Delete(false)
        End If
    Next

    Dim subFolder
    For Each subFolder in Folder.SubFolders
        DeleteExtraneousSharePointAssemblies subFolder
    Next
End Sub
```

While these copied files may not seem like a significant issue, if you configure
automated daily builds of your solution and subsequently copy each daily build
to a "Release Server" -- or if you have multiple branches of your solution
available on your local development VM -- then the disk space consumed by these
files can really add up.

For example, across the 7 branches of the solution from my previous project
(with some of the later branches containing 52 Visual Studio projects), a simple
search for **Microsoft.\*** returned 316 items consuming a whopping 746 MB of
disk space. This is a significant amount of space when you consider that I
typically allocate anywhere from 16-20GB for each VHD.

After running the script above, I regain roughly 750MB of wasted disk space.

The space savings are obviously much higher on the Release Server that archives
the Debug and Release outputs from each build.

Note that we were using Visual Studio 2005 on my previous project.
