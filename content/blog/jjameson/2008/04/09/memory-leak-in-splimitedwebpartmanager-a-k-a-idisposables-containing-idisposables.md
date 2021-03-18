---
title: Memory Leak in SPLimitedWebPartManager (a.k.a. IDisposables Containing IDisposables)
date: 2008-04-09T13:00:00-06:00
excerpt:
  Back in February, Roger Lamb kicked off his MSDN blog with a great post (
  SharePoint 2007 and WSS 3.0 Dispose Patterns by Example ). It provides
  numerous code samples that demonstrate memory leaks commonly produced when
  working with the SharePoint object...
aliases:
  [
    "/blog/jjameson/archive/2008/04/09/memory-leak-in-splimitedwebpartmanager-a-k-a-idisposables-containing-idisposables.aspx",
  ]
draft: true
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "Core Development", "WSS v3"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2008/04/09/memory-leak-in-splimitedwebpartmanager-a-k-a-idisposables-containing-idisposables.aspx](http://blogs.msdn.com/b/jjameson/archive/2008/04/09/memory-leak-in-splimitedwebpartmanager-a-k-a-idisposables-containing-idisposables.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Back in February, Roger Lamb kicked off
[his MSDN blog](http://blogs.msdn.com/rogerla) with a great post
([SharePoint 2007 and WSS 3.0 Dispose Patterns by Example](http://blogs.msdn.com/rogerla/archive/2008/02/12/sharepoint-2007-and-wss-3-0-dispose-patterns-by-example.aspx)).
It provides numerous code samples that demonstrate memory leaks commonly
produced when working with the SharePoint object model. Kudos to Roger for
putting this together.

Jon Quist, one of my fellow MCS team members on my current project, actually
discovered Roger's post last week and brought it to the attention of the rest of
the Development team. One of Jon's primary features on our project is Content
Migration (which, as the name implies, involves migrating content from the
customer's legacy ASP site to the new SharePoint site). Jon recently made some
critical changes to the content migration utilities that avoid running out of
memory when migrating thousands of pages. Kudos to Jon for digging into this and
finding Roger's post.

*Most* of Roger's examples are based on the (hopefully) well-known MSDN
whitepapers created by Scott Harris and Mike Ammerlaan:

- [Best Practices: Using Disposable Windows SharePoint Services Objects](http://msdn2.microsoft.com/en-us/library/aa973248.aspx)
- [Best Practices: Common Coding Issues When Using the SharePoint Object Model](http://msdn2.microsoft.com/en-us/library/bb687949.aspx)

We've known about the need to dispose `SPSite` and `SPWeb` objects since the
days of WSS v2 and SPS 2003 and -- generally speaking -- our custom code wraps
these instances in `using` blocks to ensure the objects are properly disposed.
However, until Jon's discovery of Roger's post, both Jon and I were perplexed by
the memory leaks in the content migration utilities. No matter how many times we
scrutinized the content migration code, we just couldn't seem to see where we
were leaking memory (and, quite honestly, it was easier to just "punt" this and
simply restart the content migration process after the occasional
`OutOfMemoryException` -- since the migration code is "smart enough" to skip
previously migrated content).

It turns out that the primary cause of memory leaks in our content migration
utilities is the first example in Roger's post:
**Microsoft.SharePoint.WebPartPages.SPLimitedWebPartManager**; "and therein, as
the Bard would tell us, lies the rub." [Score +5 bonus points if you can name
the movie that comes from.]

You see, the problem is that neither of the whitepapers noted above make any
mention of `SPLimitedWebPartManager`, much less the need to dispose of its
internal `SPWeb` object exposed through the `Web` property. Consequently, when I
saw Roger's `SPLimitedWebPartManager` example...well, let's just say that
something didn't feel "quite right" about it.

This morning I did a quick search for "SPLimitedWebPartManager dispose" on
Windows Live, and found the following:

{{< reference
title="Napier, Bryan (2007). SPLimitedWebPartManager Memory Leak? .. of ones and zeros.. 2007-06-05."
linkHref="http://blog.ofonesandzeros.com/2007/06/05/splimitedwebpartmanager-memory-leak/" >}}

I, for one, agree with Bryan's assessment. The memory leak is inherently in
`SPLimitedWebPartManager`. While it is true that Roger's example shows one way
of fixing the memory leak, the real fix -- at least in my opinion -- should be
to modify `SPLimitedWebPartManager` to dispose of its resources when it, itself,
is disposed. Heck, I'll even go so far as to say that all SharePoint classes
should be modified to behave like this. For example, `SPSite.ParentWeb` and
`SPWeb.RootWeb` should be inherently disposed by `SPSite` and `SPWeb` --
assuming the corresponding members were indeed instantiated -- instead of
relying on the caller to dispose of these.

After all, as in the case of `SPLimitedWebPartManager`, how is the caller
supposed to *always* know when the internal objects are instantiated and
consequently need to be disposed?

In general -- again, just speaking my opinion here -- if a class that implements
`IDisposable`, in turn, exposes `IDisposable` resources, then the containing
class should dispose of the underlying objects -- instead of relying on the
caller.

Unfortunately, depending on how custom code is actually written, this would
likely cause "bugs" in custom code if the SharePoint team were to actually
change this now. However, in my opinion, if you access an "inner" `IDisposable`
object exposed by an "outer" object that implements `IDisposable`, then you had
better not be using the "inner" object after disposing of the "outer" object. If
your code currently does this, then, at least IMHO, your code is just plain
wrong.

Don't agree with me? See something I am missing? Feel free to "flame me" with
comments below. Don't worry, I can take it ;-)
