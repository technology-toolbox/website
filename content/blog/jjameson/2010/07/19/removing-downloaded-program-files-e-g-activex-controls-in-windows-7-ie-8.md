---
title: "Removing Downloaded Program Files (e.g. ActiveX Controls) in Windows 7/IE 8"
date: 2010-07-19T15:13:00-06:00
excerpt:
  "In the latest sprint on my current project, we are adding yet another major
  feature to a customer service portal -- specifically, the ability to view live
  video feeds from security cameras. The vendor this particular customer has
  selected for providing..."
aliases:
  [
    "/blog/jjameson/archive/2010/07/19/removing-downloaded-program-files-e-g-activex-controls-in-windows-7-ie-8.aspx",
  ]
draft: true
categories: ["Infrastructure", "Development"]
tags: ["Windows Server", "Web Development", "Windows 7"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/07/19/removing-downloaded-program-files-e-g-activex-controls-in-windows-7-ie-8.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/07/19/removing-downloaded-program-files-e-g-activex-controls-in-windows-7-ie-8.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

In the latest sprint on my current project, we are adding yet another major
feature to a customer service portal -- specifically, the ability to view live
video feeds from security cameras. The vendor this particular customer has
selected for providing the camera functionality utilizes several ActiveX
controls to implement video feeds. Consequently, one of the processes that we
need for this sprint is the ability to remove the ActiveX controls (for example,
to test newer versions of the components, or to demonstrate the user experience
the first time someone launches the new "remote monitoring" feature in the
portal).

While you might think that simply deleting the temporary files in Internet
Explorer (something we do very frequently during development and testing) would
remove these files, that's not the case.

Also note the following KB article is effectively worthless these days even
though it includes a section for "Internet Explorer 4.x or Later (All
Platforms)":

{{< reference title="How to Remove an ActiveX Control in Windows"
linkHref="http://support.microsoft.com/kb/154850" >}}

[On a side note, I strongly recommend you think twice before using the phrase
"or later" when referring to software. Think about it...it is highly unlikely
that something you say about your current version of software will apply to all
subsequent versions of that software.]

To view the list of downloaded program files (e.g. ActiveX controls) in Internet
Explorer:

1. Open Internet Explorer, click **Tools** and then click **Internet Options**.
2. In the **Internet Options** window, on the **General** tab, in the **Browsing
   history** section, click **Settings**.
3. In the **Temporary Internet Files and History Settings** window, click **View
   Objects**.

Some Microsoft KB articles suggest that you can easily remove a downloaded
program file using the "occache.dll" shell extension simply by right-clicking
the file and then clicking "Remove". That might very well have been true back in
the old days (e.g. Windows XP). However, in my recent experience (on Windows 7
and Windows Server 2008), the only item that appears on the context menu is
"Properties" -- which doesn't provide any way to remove the item.

To remove a downloaded program file (e.g. an ActiveX control) on Windows 7 with
Internet Explorer 8:

1. Open Internet Explorer, click **Tools** and then click **Internet Options**.
2. In the **Internet Options** window, on the **Programs** tab, click **Manage
   add-ons**.
3. In the **Manage Add-ons** window, in the **Show:** dropdown list, click
   **Downloaded controls**, right-click the item that you want to remove, and
   then click **More Information**.
4. In the **More Information** window, click **Remove**. If necessary, enter the
   credentials for an administrator on the computer, and then click **Yes** to
   remove the downloaded program file.

These steps should also work for Windows Vista as well (although I haven't tried
them) -- but, honestly, does anyone even care about Windows Vista anymore? ;-)
