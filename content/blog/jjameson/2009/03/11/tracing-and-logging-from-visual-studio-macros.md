---
title: "Tracing and Logging from Visual Studio Macros"
date: 2009-03-11T01:57:00+08:00
excerpt: "As I mentioned in a post last week, I often use macros in Visual Studio to automate development tasks. 
 Before sharing some of my most frequently used macros, however, I wanted to first introduce the method I use to trace events and log messages while..."
draft: true
categories: ["Development"]
tags: ["Core Development", "Visual Studio"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/11/tracing-and-logging-from-visual-studio-macros.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/11/tracing-and-logging-from-visual-studio-macros.aspx)
> 
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

As I mentioned in a [post](/blog/jjameson/2009/03/06/large-visual-studio-solutions-by-loading-unloading-projects) last week, I often use macros in Visual Studio to automate development  tasks.

Before sharing some of my most frequently used macros, however, I wanted to first  introduce the method I use to trace events and log messages while running various  macros.

Take a look at the following **Output** window from Visual Studio.  Notice how there is an item in the **Show output from** dropdown list  titled **Macros** (a.k.a. "the macro output pane").

![Macro output pane](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Macro%20Output%20Pane.png)

    Figure 1: Macro output pane

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Macro%20Output%20Pane.png)

Chances are that when you look at your Visual Studio environment, you won't see  this item. So, why does it appear in my environment?

The macro output pane is actually created on-the-fly, as necessary, whenever  I run one of my macros that implements tracing (i.e. writes output). This is done  via the `GetMacroOutputPane()` function, as shown below.

```
Private Function GetMacroOutputPane() As OutputWindowPane
        Dim ow As OutputWindow = _
            DTE.Windows.Item(Constants.vsWindowKindOutput).Object()

        Dim outputPane As OutputWindowPane

        Try
            outputPane = ow.OutputWindowPanes.Item("Macros")
        Catch ex As Exception
            outputPane = ow.OutputWindowPanes.Add("Macros")
        End Try

        Return outputPane
    End Function
```

Pretty simple, eh? If the macro output pane exists, then use it; otherwise add  a new pane.

In order to simplify writing output messages -- as well as timestamp each message  as it is written -- I use the `WriteOutput()` method

```
Private Sub WriteOutput( _
        ByVal s As String)

        Dim buffer As StringBuilder = New StringBuilder

        buffer.Append(Date.Now.ToLongTimeString())
        buffer.Append(" ")
        buffer.Append(s)
        buffer.Append(vbCrLf)

        Dim output As String = buffer.ToString()

        Dim outputPane As OutputWindowPane = GetMacroOutputPane()
        outputPane.OutputString(output)
    End Sub
```

