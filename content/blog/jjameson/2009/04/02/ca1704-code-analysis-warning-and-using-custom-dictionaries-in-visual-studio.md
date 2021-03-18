---
title: CA1704 Code Analysis Warning and Using Custom Dictionaries in Visual Studio
date: 2009-04-02T09:16:00-06:00
excerpt:
  In my previous post , I introduced the concept of linking files in Visual
  Studio solutions. 
   A good use of this feature is specifying a custom dictionary for your
  solution. 
   Once you enable Code Analysis on your projects, you are likely to
  encounter...
aliases:
  [
    "/blog/jjameson/archive/2009/04/01/ca1704-code-analysis-warning-and-using-custom-dictionaries-in-visual-studio.aspx",
    "/blog/jjameson/archive/2009/04/02/ca1704-code-analysis-warning-and-using-custom-dictionaries-in-visual-studio.aspx",
  ]
draft: true
categories: ["My System", "Development"]
tags: ["My System", "Core Development", "Visual Studio"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/04/02/ca1704-code-analysis-warning-and-using-custom-dictionaries-in-visual-studio.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/04/02/ca1704-code-analysis-warning-and-using-custom-dictionaries-in-visual-studio.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

In my
[previous post](/blog/jjameson/2009/04/02/linked-files-in-visual-studio-solutions),
I introduced the concept of linking files in Visual Studio solutions.

A good use of this feature is specifying a custom dictionary for your solution.

Once you enable Code Analysis on your projects, you are likely to encounter
warnings similar to the following:

{{< blockquote "font-italic" >}}

MSBUILD : warning : CA1704 : Microsoft.Naming : Correct the spelling of
'Fabrikam' in namespace name 'Fabrikam.Demo.CoreServices.Logging'.

{{< /blockquote >}}

If you right-click one of these warnings and then click **Show Error Help**, you
will find the following:

{{< blockquote "font-italic" >}}

**How to Fix Violations**\
To fix a violation of this rule, correct the spelling of the word or add the
word to a custom dictionary named CustomDictionary.xml. Place the dictionary in
the installation directory of the tool, the project directory, or in the
directory associated with the tool under the user's profile
(%USERPROFILE%\Application Data\...).

{{< /blockquote >}}

{{< reference title="Identifiers should be spelled correctly"
linkHref="http://msdn.microsoft.com/en-us/library/bb264492.aspx" >}}

However, I don't recommend using any of these options. Here's why...

We want our custom dictionary to be applied to all projects in our solution --
not just one project. This would make us tend to believe that the first or last
option described above would be the best choice. However, we also want our
custom dictionary to be easily updated by any developer on the team, and -- as
always -- we also want to keep things as simple as possible.

Forcing each developer on your team to copy the CustomDictionary.xml file into
some folder -- and subsequently manage updates to this file on his or her local
development environment -- is certainly less than ideal.

Therefore, the best place to put CustomDictionary.xml is in source control right
alongside the solution file. Thus when any developer "gets latest" on the
solution, he or she will automatically get the latest custom dictionary. This
also ensures that new team members get everything they need on the first "get"
from source control.

So, now that we know we want CustomDictionary.xml stored side-by-side with the
Visual Studio solution file, the only thing left to do is configure Visual
Studio to use our custom dictionary for each project in the solution.

This is accomplished by adding a link in each project to CustomDictionary.xml
(in the solution folder) and subsequently configuring the following property on
the linked file:

- Build Action: **CodeAnalysisDictionary**

Now you can simply add items to the CustomDictionary.xml for the solution, as
shown below, and eliminate those CA1704 warnings.

```
<?xml version="1.0" encoding="utf-8" ?>
<Dictionary>
  <Words>
    <Recognized>
      <Word>Fabrikam</Word>
    </Recognized>
  </Words>
</Dictionary>
```

Since the file is in source control, you can obviously track changes to the file
and view the history at any point in the future.
