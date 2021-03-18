---
title: Deleting Empty Folders
date: 2009-11-03T05:36:00-07:00
excerpt:
  For the sake of this post, let's assume that you have a directory that
  contains some empty folders you want to get rid of. How the empty folders got
  there isn't important; all that matters is that you have some and you want to
  get rid of them. A few...
aliases:
  [
    "/blog/jjameson/archive/2009/11/02/deleting-empty-folders.aspx",
    "/blog/jjameson/archive/2009/11/03/deleting-empty-folders.aspx",
  ]
draft: true
categories: ["My System"]
tags: ["My System"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/11/03/deleting-empty-folders.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/11/03/deleting-empty-folders.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

For the sake of this post, let's assume that you have a directory that contains
some empty folders you want to get rid of. How the empty folders got there isn't
important; all that matters is that you have some and you want to get rid of
them.

A few years ago, I created the following script (starting from a sample I found
in the
[Script Center on TechNet](http://technet.microsoft.com/en-us/scriptcenter/default.aspx))
to recursively enumerate a folder structure, identify any empty folders, and
subsequently delete them.

```
Option Explicit

If (WScript.Arguments.Count <> 1) Then
    WScript.Echo("Usage: cscript DeleteEmptyFolders.vbs {path}")
    WScript.Quit(1)
End If

Dim strPath
strPath = WScript.Arguments(0)

Dim fso
Set fso = CreateObject("Scripting.FileSystemObject")

Dim objFolder
Set objFolder = fso.GetFolder(strPath)

DeleteEmptyFolders objFolder

Sub DeleteEmptyFolders(folder)
    Dim subfolder
    For Each subfolder in folder.SubFolders
        DeleteEmptyFolders subfolder
    Next

    If folder.SubFolders.Count = 0 And folder.Files.Count = 0 Then
        WScript.Echo folder.Path & " is empty"
        fso.DeleteFolder folder.Path
    End If
End Sub
```

As you can see, there's really nothing complex here. Nevertheless I still find
it to be a very useful script from time to time, so I thought I should share it.
I used it this morning and it occurred to me that I should throw it up on the
blog.

Just be sure to run it with
[CScript](http://msdn.microsoft.com/en-us/library/xazzc41b%28VS.85%29.aspx) --
and not the default WScript -- or you'll find the `WScript.Echo` messages rather
frustrating.
