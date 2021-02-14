---
title: "To Dispose or not to Dispose -- that is the question"
date: 2009-03-19T02:26:00+08:00
excerpt: "Last Saturday, another team member sent an email out to the team inquiring about the \"MOSS object disposal problem\" (as he termed it). 
 Essentially, he was asking if anytime he referenced the ParentWeb property on an object, whether or not he needed..."
draft: true
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "Core Development", "WSS v3"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/19/to-dispose-or-not-to-dispose-that-is-the-question.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/19/to-dispose-or-not-to-dispose-that-is-the-question.aspx)
> 
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

Last Saturday, another team member sent an email out to the team inquiring about  the "MOSS object disposal problem" (as he termed it).

Essentially, he was asking if anytime he referenced the `ParentWeb`  property on an object, whether or not he needed to call `Dispose` on `ParentWeb`.

When I read his message, and the various responses from other team members, it  brought back fond memories of memory leaks I've seen in the past -- which eventually  led to my ["IDisposables Containing IDisposables" post](/blog/jjameson/2008/04/09/memory-leak-in-splimitedwebpartmanager-a-k-a-idisposables-containing-idisposables).

The most interesting thing to come out of this email thread -- at least for me,  personally -- is the awareness that the prescriptive guidance on MSDN for disposing  SharePoint objects has recently been updated (one in January 2009 and the other  in March 2009).

In particular, the guidance around disposing of `ParentWeb` and `RootWeb` has been, well, nixed...

> **SPSite.RootWeb Property**
> 
> An earlier version of this article indicated that the calling application should
> dispose of the SPSite.RootWeb property just before disposing of the SPSite object
> that is using it. This is no longer the official guidance. The dispose cleanup
> is handled automatically by the SharePoint framework.
> 
> ...
> 
> **SPWeb.ParentWeb Property**
> **Updated Guidance**
> 
> An earlier version of this article recommended that the calling application
> should dispose of the SPWeb.ParentWeb. This is no longer the official guidance.
> The dispose cleanup is handled automatically by the SharePoint framework.

<cite>Scott Harris, et. al (2009). Best Practices: Using Disposable Windows
SharePoint Services Objects 2009-03-19.</cite>
[http://msdn.microsoft.com/en-us/library/aa973248.aspx](http://msdn.microsoft.com/en-us/library/aa973248.aspx)

Woohoo!!!!!

Folks, this is huge!

Now, if we could just eliminate the need for these whitepapers altogether ;-)

You see, this shouldn't even *be* a SharePoint issue, and thus there should  be no need for developers to read 44 pages of prescriptive guidance (29 pages in  one and 15 in the other).

Rather, we simply need to ingrain in every .NET developer's mind that when a  class implements `IDisposable`, thou shalt call `Dispose()`  on any instances of the class. Note that this must be done by the code that ultimately  "owns" the object.

For example, consider the following method from my `SharePointHelper`  class (which I've been using for years):

```
/// <summary>
    /// Finds a child Web based on the name (relative URL).
    /// </summary>
    /// <remarks>
    /// This is useful because SPWebCollection[name] does not return null
    /// or throw an exception if the specified web does not exist.
    /// </remarks>
    /// <param name="parentWeb">The parent Web to search.</param>
    /// <param name="name">The name (i.e. URL) of the child Web to find.</param>
    /// <returns>An SPWeb object (which must be disposed by the caller) if
    /// the child Web is found, otherwise null.</returns>
    public static SPWeb FindWeb(
        SPWeb parentWeb,
        string name)
    {
        if (parentWeb == null)
        {
            throw new ArgumentNullException("parentWeb");
        }

        if (name == null)
        {
            throw new ArgumentNullException("name");
        }

        name = name.Trim();
        if (string.IsNullOrEmpty(name))
        {
            throw new ArgumentException(
                "The name of the Web must be specified.",
                "name");
        }

        SPWeb childWeb = parentWeb.Webs[name];

        // HACK: SPWebCollection[name] does not return null or throw
        // an exception if the specified web does not exist.
        try
        {
            Guid webId = childWeb.ID;

            // In order to avoid a code analysis warning about webId not
            // being used we need to do "something" with webId;
            // a simple assertion is sufficient
            Debug.Assert(webId != Guid.Empty);
        }
        catch (FileNotFoundException)
        {
            childWeb.Dispose();
            childWeb = null;
        }

        return childWeb;
    }
```

As noted in the XML comments for this method, the caller is responsible for disposing  of the returned `SPWeb` -- assuming the Web was successfully found. If  the `SPWeb` was not found, then obviously `SharePointHelper`  must dispose of the (bogus) object.

So getting back to the title of this post: To `Dispose` or not to `Dispose`...

The answer should be very easy, and certainly should not comprise 44 pages.

How about one paragraph (and a code sample)?

If you instantiate an object that implements `IDisposable` -- either  through the `new` operator or through a method like the sample shown  above -- then you should call `Dispose()` on that object, or, preferably  if you are a C# developer, you should wrap your object in a `using` block,  as shown below:

```
private void ConfigureSampleContentWeb(
        SPWeb parentWeb)
    {
        using (SPWeb samplesWeb =
            SharePointHelper.FindWeb(parentWeb, "Samples"))
        {
            ConfigureSamplePages(samplesWeb);
        }
    }
```

Am I making this too easy?

Let me know your thoughts.

Although I didn't receive a single comment on my ["IDisposables Containing IDisposables" post](/blog/jjameson/2008/04/09/memory-leak-in-splimitedwebpartmanager-a-k-a-idisposables-containing-idisposables), I have to wonder if lots of people  out there aren't silently agreeing with this concept. There were certainly some  comments to [Roger's post](http://blogs.msdn.com/rogerla/archive/2008/02/12/sharepoint-2007-and-wss-3-0-dispose-patterns-by-example.aspx) that said effectively the same thing.

I completely understand that all software has bugs (heck, just read [my previous post](/blog/jjameson/2009/03/19/argumentnullexception-with-optional-publishingpage-description-property-with-some-thoughts-on-breaking-the-build-too) if you want to see a rather embarrassing personal example of  one of my own) and that it takes time to fix bugs. However, we should always strive  to make the code we write the best -- and simplest -- that it can possibly be.

Now, if we could just get that all-encompassing FxCop rule for ensuring that  all `IDisposable` objects are wrapped in `using` blocks...  ;-)

