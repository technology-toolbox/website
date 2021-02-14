---
title: "ArgumentNullException with Optional PublishingPage.Description Property (with some thoughts on breaking the build, too)"
date: 2009-03-19T01:06:00+08:00
excerpt: "Yesterday morning I broke the build. 
 Ouch. 
 Technically speaking, the changes that I checked in did not break the build, per se, but rather my changes caused a nasty ArgumentNullException while redeploying our SharePoint site. 
 In \"Developerspeak..."
draft: true
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "Core Development"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/19/argumentnullexception-with-optional-publishingpage-description-property-with-some-thoughts-on-breaking-the-build-too.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/19/argumentnullexception-with-optional-publishingpage-description-property-with-some-thoughts-on-breaking-the-build-too.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

Yesterday morning I broke the build.

Ouch.

Technically speaking, the changes that I checked in did not break the build, per se, but rather my changes caused a nasty `ArgumentNullException` while redeploying our SharePoint site.

In "Developerspeak", breaking the build means you either checked in a change that causes the actual code compilation to fail (which is practically inexcusable, but nevertheless still seems to happen from time to time), or causes one of the BVTs (i.e. Build Verification Tests) to fail.

In other words, if you can't successfully create a SharePoint site due to an `ArgumentNullException` during feature activation, then the "build" is definitely broken.

And to think, I almost got away with it.

We were pushing for a 12:00 PM EDT cutoff for this milestone. I was working on some important changes to my Search feature and I was rushing to get them completed and included in this build. Essentially I was simply trying to show the page descriptions in search results instead of the `HitHighlightedSummary` rendered by default in Microsoft Office SharePoint Server (MOSS) 2007. It seemed like a pretty trivial change and the unit tests I was running to validate my changes were, indeed, showing green.

The problem is, I wasn't running *all* of the unit tests that I needed to. Specifically, I was only running tests that actually specified a page description.

When I checked in my changeset at roughly 11:38 AM, I thought I was all set for the cutoff a few minutes later. However, after getting latest on the entire solution and rebuilding my Web application (a lengthy process, but fortunately scripted, so it doesn't require any significant effort), I encountered the exception.

At that point, I almost started to panic.

Fortunately, by 11:54 AM, my updated changeset was checked in and the bug was fixed -- with 6 minutes to spare! However, as I mentioned before I *almost* got away with it.

It turns out that Dan, another developer on the team, apparently got latest sometime during that 16-minute epoch, and started rebuilding his Web app.

When he sent out an email to the team, I was forced to Reply All, fess up about the bug, apologize, and tell him to immediately get latest and try again.

So, lessons learned...

I typically create top-level test lists in Visual Studio called **Long Running Tests** and **Quick Tests**. Well, you can guess which tests I ran before checking in my changeset ;-)

Just remember, after you have unit tested your changes, subsequently diff'ed the code to ensure no errant changes were made, and then proceed guiding the cursor rapidly towards that **Check In** button...slow down, take a deep breath, and validate whether you really feel confident that you haven't broken anything. This is especially important when you are "under the gun" from a time perspective -- when obviously mistakes are more likely.

Perhaps those "Long Running Tests" aren't really so long after all. I know the time that my bug cost Dan was far more than the time it would have taken me to run all of the unit tests prior to checking in.

You may be wondering, what was the bug? It seems laughable, but you can't set [PublishingPage.Description](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.publishing.publishingpage.description.aspx) to null. You can set it to an empty string -- just not null. Go figure.

I know, I know...the previous link shows that this is clearly the documented behavior, but still...from an API perspective, perhaps there's another lesson here.

If a string property on an object is optional (e.g. the description of a page) -- which presumably means it is stored as NULL in the database -- then don't throw an `ArgumentNullException` when someone passes in a null string, and instead force them to pass in `string.Empty`. It's just not intuitive -- or am I the only developer who thinks this?

Let me know if you agree or disagree.

