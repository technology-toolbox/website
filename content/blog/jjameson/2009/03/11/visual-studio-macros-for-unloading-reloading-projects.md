---
title: "Visual Studio Macros for Unloading/Reloading Projects"
date: 2009-03-11T02:30:00+08:00
excerpt: "As promised in a post last week, here are the macros that I use to quickly unload or reload dozens of projects in a large Visual Studio solution. Hmmm, perhaps effortlessly is a better word choice -- considering I might need to wait 30 seconds or so for..."
draft: true
categories: ["Development"]
tags: ["Core Development", "Visual Studio"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/11/visual-studio-macros-for-unloading-reloading-projects.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/11/visual-studio-macros-for-unloading-reloading-projects.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

As promised in a [post](/blog/jjameson/2009/03/06/large-visual-studio-solutions-by-loading-unloading-projects) last week, here are the macros that I use to quickly unload or reload dozens  of projects in a large Visual Studio solution. Hmmm, perhaps *effortlessly*  is a better word choice -- considering I might need to wait 30 seconds or so for  all of the projects to unload or reload.

Here is my `UnloadAllProjects()` macro:

```
Public Sub UnloadAllProjects()
    If (DTE.Solution Is Nothing _
        Or DTE.Solution.Count = 0) Then

        MessageBox.Show( _
            "Cannot unload all projects because no solution is open.", _
            "Error", _
            MessageBoxButtons.OK, _
            MessageBoxIcon.Error)

        Exit Sub
    End If

    Dim solutionName As String = _
        DTE.Solution.Properties.Item("Name").Value.ToString()

    WriteOutput("Unloading projects in solution (" & solutionName & ")...")

    Dim solutionExplorer As Window = _
        DTE.Windows.Item(Constants.vsWindowKindSolutionExplorer)

    solutionExplorer.Activate()

    Dim solutionHierarchy As UIHierarchy = solutionExplorer.Object

    For Each proj As Project In DTE.Solution.Projects
        ' Note that the project may be a solution folder, in which case we
        ' want to unload all of the projects within the solution folder.
        '
        ' Also note if the project is already unloaded, or is a solution
        ' folder containing no projects -- then a COM exception occurs.
        Try
            UnloadProject(proj, solutionName, solutionHierarchy)
        Catch ex As Exception
            WriteOutput("Error unloading project (" & proj.Name & "): " _
                & ex.Message)

        End Try
    Next

    WriteOutput("Successfully unloaded projects in solution (" _
                & solutionName & ").")
End Sub
```

The actual work of unloading a project is delegated to the `UnloadProject()`  method:

```
Private Sub UnloadProject( _
    ByVal proj As Project, _
    ByVal solutionName As String, _
    ByVal solutionHierarchy As UIHierarchy)

    Debug.Assert(Not proj Is Nothing)
    Debug.Assert(String.IsNullOrEmpty(solutionName) = False)
    Debug.Assert(Not solutionHierarchy Is Nothing)

    WriteOutput("Unloading project (" & proj.Name & ")...")

    If (proj.Kind = Constants.vsProjectKindMisc) Then
        ' "Miscellaenous Files" cannot be unloaded
        WriteOutput("Project (" & proj.Name & ") cannot be unloaded.")
        Return
    End If

    Dim projPath As String = solutionName & "\" & proj.Name

    Dim obj As Object = solutionHierarchy.GetItem(projPath)
    obj.Select(vsUISelectionType.vsUISelectionTypeSelect)

    Dim projectName As String = proj.Name
    DTE.ExecuteCommand("Project.UnloadProject")
    WriteOutput("Successfully unloaded project (" & projectName & ").")
End Sub
```

A few interesting notes...

- I originally used the **Record TemporaryMacro** feature in
  Visual Studio a few years ago to determine how projects are loaded and unloaded.
  This led me to the scantily documented `DTE.ExecuteCommand("Project.UnloadProject")`.
- I discovered that a specific project type ("Miscellaneous Files") will always
  throw a COM exception whenever you try to unload or reload it. Consequently,
  I skip that one altogether.
- While I tried to find a way to determine if a particular project (or projects
  within a solution folder) are already unloaded -- and thus avoid a COM exception
  when subsequently attempting to unload it again -- I ended up punting this and
  instead simply wrapping the "unload" operation in a try/catch block.
- The `WriteOutput()` method is provided in
  [a previous post](/blog/jjameson/2009/03/11/tracing-and-logging-from-visual-studio-macros).

If you take the above code and essentially search-and-replace "unload" with "reload"  then you will end up with my macro to reload all projects within the solution:

```
Public Sub ReloadAllProjects()
    If (DTE.Solution Is Nothing _
        Or DTE.Solution.Count = 0) Then

        MessageBox.Show( _
            "Cannot reload all projects because no solution is open.", _
            "Error", _
            MessageBoxButtons.OK, _
            MessageBoxIcon.Error)

        Exit Sub
    End If

    Dim solutionName As String = _
        DTE.Solution.Properties.Item("Name").Value.ToString()

    WriteOutput("Reloading projects in solution (" & solutionName & ")...")

    Dim solutionExplorer As Window = _
        DTE.Windows.Item(Constants.vsWindowKindSolutionExplorer)

    solutionExplorer.Activate()

    Dim solutionHierarchy As UIHierarchy = solutionExplorer.Object

    For Each proj As Project In DTE.Solution.Projects
        ' Note that the project may be a solution folder, in which case we
        ' want to reload all of the projects within the solution folder.
        '
        ' Also note if the project is already loaded, or is a solution
        ' folder containing no projects -- then a COM exception occurs.
        Try
            ReloadProject(proj, solutionName, solutionHierarchy)
        Catch ex As Exception
            WriteOutput("Error reloading project (" & proj.Name & "): " _
                & ex.Message)

        End Try
    Next

    WriteOutput("Successfully reloaded projects in solution (" _
                & solutionName & ").")
End Sub

Private Sub ReloadProject( _
    ByVal proj As Project, _
    ByVal solutionName As String, _
    ByVal solutionHierarchy As UIHierarchy)

    Debug.Assert(Not proj Is Nothing)
    Debug.Assert(String.IsNullOrEmpty(solutionName) = False)
    Debug.Assert(Not solutionHierarchy Is Nothing)

    WriteOutput("Reloading project (" & proj.Name & ")...")

    If (proj.Kind = Constants.vsProjectKindMisc) Then
        ' "Miscellaenous Files" cannot be unloaded
        WriteOutput("Project (" & proj.Name & ") cannot be unloaded.")
        Return
    End If

    Dim projPath As String = solutionName & "\" & proj.Name

    Dim obj As Object = solutionHierarchy.GetItem(projPath)
    obj.Select(vsUISelectionType.vsUISelectionTypeSelect)

    DTE.ExecuteCommand("Project.ReloadProject")
    WriteOutput("Successfully reloaded project (" & proj.Name & ").")
End Sub
```

Like I mentioned earlier, unloading or reloading all of the projects within a  solution is not exactly an instantaneous operation. However, when you consider the  incremental build time saved by unloading all of the projects in a large solution  except for the few you are working on at any given time, you definitely come out  far ahead in the end. You'll be amazed at how much more productive you can be when  your solution containing 50+ projects incrementally builds in just a few seconds  when all but a handful of the projects are unloaded.

I hope you find these macros as helpful as I have over the years.

Perhaps in a future version of Visual Studio, we'll have the ability to right-click  the solution in **Solution Explorer** and then click something like **Unload Projects in Solution** (similar to the **Unload Projects
in Solution Folder** option that is currently available -- but only for solution  folders, not the solution itself). Until then, I don't see me giving up my dependency  on these macros anytime soon.

