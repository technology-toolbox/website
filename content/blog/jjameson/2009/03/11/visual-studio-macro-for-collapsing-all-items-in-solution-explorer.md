---
title: "Visual Studio Macro for Collapsing All Items in Solution Explorer"
date: 2009-03-11T03:18:00+08:00
excerpt: "Along with my Visual Studio macros for unloading/reloading projects in a solution , another macro that I use just as much, if not more frequently, is my CollapseAllItems() macro: 
 
 Public Sub CollapseAllItems()
 Dim solutionExplorer As Window = _..."
draft: true
categories: ["Development"]
tags: ["Core Development", "Visual Studio"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/11/visual-studio-macro-for-collapsing-all-items-in-solution-explorer.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/11/visual-studio-macro-for-collapsing-all-items-in-solution-explorer.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

Along with [my Visual Studio macros for unloading/reloading projects in a solution](/blog/jjameson/2009/03/11/visual-studio-macros-for-unloading-reloading-projects), another  macro that I use just as much, if not more frequently, is my `CollapseAllItems()`  macro:

```
Public Sub CollapseAllItems()
    Dim solutionExplorer As Window = _
        DTE.Windows.Item(Constants.vsWindowKindSolutionExplorer)

    DTE.SuppressUI = True

    Try
        Dim solutionHierarchy As UIHierarchy = solutionExplorer.Object

        For Each item As UIHierarchyItem _
            In solutionHierarchy.UIHierarchyItems

            CollapseItem(item, solutionHierarchy)
        Next
    Catch ex As Exception
        WriteOutput("Error collapsing all items: " _
            & ex.Message)

    Finally
        DTE.SuppressUI = False
    End Try
End Sub
```

The `CollapseItem()` method is used to recursively collapse each item  in the hierarchy:

```
Private Sub CollapseItem( _
    ByVal item As UIHierarchyItem, _
    ByVal solutionHierarchy As UIHierarchy)

    For Each child As UIHierarchyItem In item.UIHierarchyItems
        CollapseItem(child, solutionHierarchy)
    Next

    If (item.UIHierarchyItems.Expanded = True) Then
        WriteOutput("Collapsing item (" & item.Name & ")...")
        item.UIHierarchyItems.Expanded = False

        ' HACK: Known bug in Visual Studio 2005
        ' http://connect.microsoft.com/VisualStudio/feedback/ViewFeedback.aspx?FeedbackID=114597
        If (item.UIHierarchyItems.Expanded = True) Then
            item.Select(vsUISelectionType.vsUISelectionTypeSelect)
            solutionHierarchy.DoDefaultAction()
        End If
    End If
End Sub
```

> **Update (2010-08-25)**
>
> In my original post, the `If`block in **CollapseItem** was mistakenly nested inside
> the `For Each`loop. While
> this worked (I've been using it that way for years), it certainly wasn't
> optimal and, more importantly, it also was the source of some confusion
> (see Keith Robertson's comment on this post).

While it is great that Visual Studio "synchronizes" the **Solution Explorer**  window to show the current file in the solution hierarchy, in large Visual Studio  solutions, things can get a bit bewildering at times if many of the projects are  expanded down to the level of individual files.

Perhaps in a future version of Visual Studio, we'll have the ability to right-click  the solution in **Solution Explorer** and then click something like **Collapse All**. Until then, I don't see me giving up my dependency  on this macro anytime soon.

