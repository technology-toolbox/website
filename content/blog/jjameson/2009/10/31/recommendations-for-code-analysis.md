---
title: Recommendations for Code Analysis
date: 2009-10-31T05:44:00-06:00
excerpt:
  In my previous post , I briefly mentioned the Code Analysis feature of Visual
  Studio in the context of using check-in policies with Team Foundation Server
  (TFS). However, there's a lot more to talk about with regards to using Code
  Analysis. If you...
aliases:
  [
    "/blog/jjameson/archive/2009/10/30/recommendations-for-code-analysis.aspx",
    "/blog/jjameson/archive/2009/10/31/recommendations-for-code-analysis.aspx",
  ]
draft: true
categories: ["My System", "Development"]
tags: ["My System", "Core Development", "Visual Studio"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/10/31/recommendations-for-code-analysis.aspx"
---

In my
[previous post](/blog/jjameson/2009/10/31/recommended-check-in-policies-for-team-foundation-server),
I briefly mentioned the Code Analysis feature of Visual Studio in the context of
using check-in policies with Team Foundation Server (TFS). However, there's a
lot more to talk about with regards to using Code Analysis.

If you are ever find yourself "starting from a clean slate" on a software
development project, I hope you take advantage of the opportunity to not only
enable Code Analysis, but also to treat all Code Analysis warnings as errors.
That's right, go ahead and check all of those boxes (for both **Debug** and
**Release** configurations, of course) so that whenever potential issues are
reported by Code Analysis, it stops developers dead in their tracks and
considers the build to be broken.

While you're at it, go ahead and select the **All** option in the **Treat
warnings as errors** section on the **Build** tab (something I definitely
recommend as a best practice). Again, be sure to do this for all build
configurations.

Yes, this is going to be really painful for some developers.

For example, you can say goodbye to using the overload of string.Format() that
doesn't specify a CultureInfo:

```JavaScript
            string foo = "here";

            string logMessage = string.Format(
                "Imagine something interesting {0}.",
                foo);
```

Attempting this will now result in a broken build:

{{< blockquote "fst-italic text-danger" >}}

Error 2 CA1305 : Microsoft.Globalization : Because the behavior of
'string.Format(string, object)' could vary based on the current user's locale
settings, replace this call in 'Program.Main(string[])' with a call to
'string.Format(IFormatProvider, string, params object[])'. If the result of
'string.Format(IFormatProvider, string, params object[])' will be displayed to
the user, specify 'CultureInfo.CurrentCulture' as the 'IFormatProvider'
parameter. Otherwise, if the result will be stored and accessed by software,
such as when it is persisted to disk or to a database, specify
'CultureInfo.InvariantCulture'.

{{< /blockquote >}}

To avoid the CA1305 error, you will need to use something like this instead:

```JavaScript
            string foo = "here";

            string logMessage = string.Format(
                CultureInfo.InvariantCulture,
                "Imagine something interesting {0}.",
                foo);
```

You will also need to enable signing of all assemblies with a strong name key,
and also specify various assembly level attributes (e.g.
[`\[assembly: CLSCompliant(true)\]`](http://msdn.microsoft.com/en-us/library/system.clscompliantattribute.aspx)).
However, if you follow the steps I've provided in a
[previous post](/blog/jjameson/2009/04/03/shared-assembly-info-in-visual-studio-projects),
this should only take you at most 1-2 minutes.

Like I said before, this will probably cause some heartburn in the beginning,
but trust me, developers will quickly get used to it and the code they write
will be much better as a result.

Now what about the other 99% of the time when you aren't starting from a clean
slate, but instead are dealing with a substantial code base that causes the Code
Analysis feature in Visual Studio to "light up like a Christmas tree" when you
turn it on?

Well, as with all software bugs, it all comes down to *triage*. You obviously
cannot simply halt all feature development in order to resolve code analysis
issues. Instead, you should go for the "low hanging fruit" (meaning the issues
that occur frequently but can be resolved with relatively little effort or risk
of changing). You should also try to enable the Code Analysis rules that are
most important, such as potential security issues.

The following post provides some great information about which Code Analysis
warnings to focus your attention on first:

{{< reference
title="What rules do Microsoft have turned on internally? 2007-08-09."
linkHref="http://blogs.msdn.com/fxcop/archive/2007/08/09/what-rules-do-microsoft-have-turned-on-internally.aspx" >}}

This article is obviously a little dated, but still very relevant over two years
later.

Note that you will undoubtedly need to
[suppress some specific Code Analysis warnings](http://msdn.microsoft.com/en-us/library/ms244717.aspx)
from time to time, and therefore you'll have to choose how to suppress the
warnings (whether inline within the source, or in a GlobalSuppressions.cs file).
However, I hope you'll choose to fix the underlying issue that causes the Code
Analysis warning (whenever practical) instead of simply suppressing a bunch of
warnings.

So, in conclusion, think of the Code Analysis feature in Visual Studio as a way
of *choosing* a more strict compiler. For example, back in the days when I used
to program in C on Unix, the compiler wouldn't complain when I wrote something
like this (even though this is obviously very wrong):

```JavaScript
            if (i = 1)
            {
                ...
            }
```

I was relieved when I switched to C++ and these kinds of bugs were a thing of
the past. [Note that I never switch my coding style to use something like "`1 == i`" in order to avoid these errors.]

Fortunately, in C# you don't have a choice about whether or not this compiles
(it doesn't).

I like to think of enabling all of the Code Analysis rules in a similar fashion.
In other words, I want the compiler to tell me whenever I am doing something
potentially wrong, because then I can spend just a little bit of time fixing my
code to do it right. By forcing the Code Analysis warnings as errors, I don't
have a choice -- meaning I have to fix them before check-in.
