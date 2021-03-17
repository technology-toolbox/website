---
title: "Why I chose not to programmatically filter errors reported by ELMAH"
date: 2012-02-28T22:24:38-07:00
excerpt: "If you think that programmatically filtering errors in ELMAH is as easy as specifying filters in Web.config, you might be in for a surprise."
aliases: ["/blog/jjameson/archive/2012/02/28/why-i-chose-not-to-programmatically-filter-errors-reported-by.aspx"]
draft: true
categories: ["Development"]
tags: ["Core Development", "Web Development"]
---

If you've been following along for the last couple of posts related to ELMAH --
or if you've read one of the
[two](http://code.google.com/p/elmah/issues/detail?id=277)
[issues](http://code.google.com/p/elmah/issues/detail?id=278) (i.e. bugs) I
created on the [ELMAH project](http://code.google.com/p/elmah) -- then you might
be wondering:

> Why didn't Jeremy just "punt" the configuration-based filtering (after
> discovering the issues) and instead create an ELMAH filter in code (i.e.
> programmatically)?

Well, I actually tried that approach, but I didn't get very far down that road
before encountering another blocking issue.

As soon as I added a "`using Elmah;`" declaration to my code-behind file for
Global.asax, I discovered the Elmah assembly is not signed. Since the "Caelum"
assemblies (i.e. the code running TechnologyToolbox.com) are all signed with a
strong name key, any assemblies *explicitly referenced at build time* must be
signed as well.

Up to that point, I had gotten by using the unsigned Elmah.dll since it was
previously referenced only in the Web.config file.

After a quick search of the issues list for ELMAH, I found
[issue 109 (Strong name the assembly)](http://code.google.com/p/elmah/issues/detail?id=109).
Shortly thereafter, I stumbled upon a discussion thread which, quite honestly,
made my head hurt (in one of those morning after New Year's Eve sort of ways):

{{< reference title="The assembly strong naming conundrum"
linkHref="http://nuget.codeplex.com/discussions/247827" >}}

> **Warning**
>
> Click the link above at your own peril.

Since Microsoft recommends signing assemblies as a best practice (there's even
an FxCop rule for it), I've been signing .NET assemblies for as long as I can
remember (regardless of whether or not the assemblies need to be loaded into the
GAC).

Not wanting to compile my own ELMAH assembly, I quickly retreated from trying to
filter errors programmatically.

I do hope the whole NuGet "strong naming conundrum" gets solved (soon) -- but to
be honest, I'm glad that I'm not involved in that mess ;-)

Personally, I'm sticking with my "Thou shalt sign all .NET assemblies"
commandment (until someone can convince me otherwise).
