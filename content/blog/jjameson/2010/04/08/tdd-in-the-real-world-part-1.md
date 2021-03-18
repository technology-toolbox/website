---
title: TDD in the Real World, Part 1
date: 2010-04-08T19:40:00-06:00
excerpt:
  "Earlier today I presented a \"Knowledge Transfer\" session to a team of
  developers on my current project. If you've ever worked with consultants,
  you've probably experienced a \"KT\" session or something similar. In essence,
  it's just a meeting intended to..."
aliases:
  [
    "/blog/jjameson/archive/2010/04/08/tdd-in-the-real-world-part-1.aspx",
  ]
draft: true
categories: ["Development"]
tags: ["Core Development"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/04/08/tdd-in-the-real-world-part-1.aspx"
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/08/tdd-in-the-real-world-part-1.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/08/tdd-in-the-real-world-part-1.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Earlier today I presented a "Knowledge Transfer" session to a team of developers
on my current project. If you've ever worked with consultants, you've probably
experienced a "KT" session or something similar. In essence, it's just a meeting
intended to cover one particular feature area of the solution and provide an
opportunity for the "maintenance" team (i.e. people responsible for supporting
and enhancing the solution going forward) to ask questions of the "development"
team (i.e. the people who built the feature).

One of the topics I listed in the agenda for the KT session was around unit
testing and Test Driven Development (TDD). This topic was of particular interest
to the audience, which really wasn't surprising considering it was also a hot
topic at a session I presented at Microsoft TechReady over a year ago.

I'll be the first to admit that I wasn't exactly an early adopter of TDD (at
least not the principles we typically associate with TDD today). Sure, I've been
creating unit tests for practically as long as I can remember writing code.
Okay, not quite...but I can honestly remember creating XML input and output
files that served effectively as unit tests over ten years ago while I was at
Micromedex working with C++ and CORBA.

I started using NUnit years ago, but sometime around 2005 I switched over to the
Visual Studio tools (for obvious reasons). I recall an internal training session
presented by Martin Fowler in which he explained the concept of writing the test
first, ensuring it fails, and then writing the code to make the test pass.

When I first started using NUnit, I kept thinking to myself: "This is
great!...However, while it's really easy to do TDD when you are writing
something simple like a Calculator service, it's often difficult to do it in the
real world." Even after reading
[Extreme Programming Adventures in C#](http://www.microsoft.com/learning/en/us/book.aspx?ID=6777&locale=en-us)
a few years ago -- which is a very good book, by the way -- I still find it
difficult sometimes to use TDD when creating enterprise solutions with numerous
external dependencies (backend databases, SharePoint, etc.).

It's been well over 5 years since I worked on a "pure" .NET solution; rather
most of my time is spent working on solutions based on SharePoint, and much less
frequently these days, BizTalk Server. I also get called in occasionally to do
something spiffy with SQL Server and Analysis Services.

Sometimes I dream about how nice it would be to just build something with
nothing more than what comes out-of-the-box with Visual Studio and the .NET
Framework. Things like ASP.NET MVC just seem so developer-friendly. Of course,
then I start thinking about all the really cool things you can do with
SharePoint -- even if it does make life more, um, *challenging* in some regards.

The way I approach TDD is to do it where it makes sense, meaning that it adds
more value over the long run than the effort you put into it. Many people might
argue that TDD should be used all the time, but I doubt those people spend most
of their time working on SharePoint solutions ;-)

Last year I watched a recorded webcast about doing TDD with SharePoint, which
focused heavily on using Typemock to workaround the fact that the SharePoint API
isn't TDD-friendly in the slightest. Note that I have essentially zero
experience at present with any of the mocking frameworks out there. Sure, I've
heard of Typemock, RhinoMock, and Moq -- and seen plenty of code samples
demonstrating their use -- but there's just something about them that doesn't
feel right to me.

While I certainly like the concept of creating true "unit tests" -- instead of
"integration tests" -- and isolating external dependencies as much as possible,
one thing kept nagging at me throughout the webcast until finally the presenter
admitted it near the end of the session...

While the "unit tests" they created with TypeMock were all passing (i.e. green),
when they subsequently ran the code within an actual SharePoint site, they found
that the solution broke in several areas and they ended up writing a different
set of "integration tests" to account for the different behavior. When I heard
that part of the webcast, it only solidified my previous perception of mocking.
Now, to be honest, I really don't know how much code churn was necessary after
integrating with the live SharePoint site (the presenter didn't say).

Were they better off as a result of using TDD and Typemock? Probably.

Was the investment in mocking the SharePoint API worth it? I certainly hope so.

The reality is that sometimes mocking is definitely the way to go -- heck, maybe
even most of the time (depending on the technology stack you are working with).
Perhaps I'm overly biased by the fact that I, personally, find the code for unit
tests that utilize one of the mock frameworks just so darn cryptic and difficult
to read. I'm sure part of this is just that I really haven't sat down and
devoted the necessary time to learn one of the frameworks. Of course, I also
find LINQ and lamda expressions much harder to read than plain ol' C# code --
but perhaps that's just me ;-)

So...it's getting late and this post isn't really going in the direction that I
originally intended. What I really wanted to do was demonstrate a couple of real
world examples of TDD that I've used, but I think I'll punt that until tomorrow
morning.

I realize that this post might serve no other purpose than as "flame bait" for
those of you out there that thrive on one of the mock frameworks that I
mentioned above. Heck, if you are successfully using Typemock or some other
mocking framework extensively with SharePoint -- and you're truly happy with the
ROI (meaning you get decent code coverage and don't encounter lots of issues in
your "green" code when you deploy it to a live SharePoint site) -- then I'd love
to hear from you. As I've said before, feel free to "flame away" -- I can take
it ;-)

Otherwise, stay tuned for part 2 of this post where I'll show you some of my
"developer tests" (note that I often mix "unit tests" and "integration tests" in
my ".DeveloperTests" projects, because in my mind, both of these types of tests
add value, but sometimes one is more appropriate than the other).
