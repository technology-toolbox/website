---
title: Dumping MOSS 2007 Variations - Part 1
date: 2007-10-30T08:56:00-06:00
excerpt:
  Shortly before I headed out to the airport last Wednesday, I received the
  fateful email from my customer notifying us that they have decided to abandon
  using the variations feature in Microsoft Office SharePoint Server (MOSS)
  2007. Note that the original...
aliases:
  [
    "/blog/jjameson/archive/2007/10/29/dumping-moss-2007-variations-part-1.aspx",
    "/blog/jjameson/archive/2007/10/30/dumping-moss-2007-variations-part-1.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/10/30/dumping-moss-2007-variations-part-1.aspx"
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2007/10/30/dumping-moss-2007-variations-part-1.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/10/30/dumping-moss-2007-variations-part-1.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Shortly before I headed out to the airport last Wednesday, I received the
fateful email from my customer notifying us that they have decided to abandon
using the variations feature in Microsoft Office SharePoint Server (MOSS) 2007.
Note that the original release date for our solution was October 19th -- and the
decision to cut variations from our solution would obviously require significant
rework (meaning we would not come close to hitting our original date). This was
obviously a difficult decision and required careful consideration to determine
how best to move forward.

I'll cover the reasons why this decision was made over a series of three posts
-- with each post detailing each of the major issues that we faced while trying
to use variations.

In part 1, I'll focus on the incompatibility of the out-of-the-box (OOTB)
variations feature and the OOTB content types feature.

We first encountered this problem about four weeks ago when we noticed that
pages were not always propagating from the source site (i.e. label ) to other
variation sites. Based on a cursory scan of the variation logs, the failures
appeared to fall into one of the following error categories:

1. The variation system failed to pair up pages
   [http://foobar/en-US/Support/FAQs/foo/Pages/default.aspx](http://foobar/en-US/Support/FAQs/foo/bar/Pages/default.aspx)
   and /ja-JP/Support/FAQs/foo/Pages/default.aspx because their Content Types do
   not match.
2. Object reference not set to an instance of an object.

The first error is due to the fact that MOSS 2007 does not automatically change
the content type of the variation page to match the source page and instead
simply throws an error. As the error message clearly states, the pages are not
"paired up" (which I infer to mean that there is no corresponding entry in the
hidden **Relationships List**) and therefore no updates are propagated to the
variation site (e.g. "ja-JP") when changes are made to a page on the source site
(in our case, "en-US"). [Make note of the hidden **Relationships List** -- it
becomes important in part 2 of this series.]

To be honest, we "punted" the errors in the second category above (i.e. those
caused by some mysterious NullReferenceException), since there weren't very many
and we had more important problems to investigate. Besides, we could always come
back to that later if we found it still occurred in subsequent builds of our
solution.

The fundamental problem is that we change the content type (and page layout) of
the default pages in order to display the hierarchy of Frequently Asked
Questions (FAQs) next to a list of the FAQs for the currently selected site
within the hierarchy. This is cleverly done using a custom page layout that one
of my fellow MCS team members, Ron Tielke, developed that contains an ASP.NET
menu control on the left and a Content Query Web Part on the right. In other
words, the content type of **default.aspx** starts out as **Welcome Page**
(since we are using the OOTB **Publishing with Workflow** site definition), but
we change it to **FAQ Node Page** (a custom content type that we have defined
which derives from **Page** -- just like **Welcome Page**). Consequently, users
can drill down into the FAQ sites and view FAQs within a particular category.

Once we understood the nature of the error, we believed that we could circumvent
the problem with custom content types by waiting until after the variation sites
and pages are propagated (and "paired up") before changing the content type. In
fact, this worked...for a while.

Note that we are migrating this customer from a legacy system and we have
developed custom content migration tools to build out the site structure in
SharePoint and bulk load about 3,200 FAQs into the MOSS-based solution.

To avoid the "Content Types do not match" problem, we modified our content
migration tool to break it down into a series of steps (which were previously
performed in a "fast-as-you-can" fashion). The "pace" of the migration would
then need to controlled by the user performing the migration:

1. Run the first step to create the FAQ sites with the default page having the
   default content type (i.e. **Welcome Page**)
2. Wait for steady state (i.e. let the SharePoint timer job finish creating
   variation sites and "pairing" up the pages)
3. Run the next step of the migration to change the default page of each and
   every FAQ site to use the custom **FAQ Node Page** content type

Last week, I had a conversation with the Program Manager who spec'ed the
Variations feature and he was quick to trump what I consider to be a SharePoint
bug with the "by design" card. He cited the fact that there are many things not
handled by the variations feature, such as lists, document libraries, etc.

I am fine with those limitations (in part because I had read about them months
ago when researching the variations feature so I knew to set some expectations
with the customer on what variations will and will not do). Acknowledging that
MOSS 2007 variations is all about Web Content Management (WCM), it is perfectly
reasonable that the propagation only include the **Pages** library (or the
**Paginas** library, as we discovered during our testing of the SharePoint
language packs -- but that's a funny story that deserves a post of its own, some
other time perhaps).

As I mentioned earlier, our workaround (i.e. modifying our content migration
tools) worked for a while, however we discovered just how "brittle" our solution
was when we attempted to create a new variation label. Yep, you guessed
it...lots of errors about "failed to pair up pages ... and ... because their
Content Types do not match."

We even contemplated creating all of the possible variation labels this customer
would need for the foreseeable future, but at some point, the insanity had to
stop (you'll hear more about the other problems we encountered in parts 2 and
3). Hence the decision to stop using the variations feature altogether.

The fundamental problem that I have with the "by design" response is that if you
can break one feature in a product by using another feature in the product, it
is either "poor design" or "poor implementation." In defense of the SharePoint
team, I believe it is the latter and I believe it was simply due to time
constraints in the original schedule (the PM indicated they discovered this
problem around the Beta 2 timeframe). I certainly hope they have every intention
of fixing this (although it is too late for my current customer) but based on my
conversation with the PM, I don't see this happening anytime soon (it is
certainly not slated for SP1).

> **Update (2007-11-28)**
>
> According to a
> [follow-up](http://blogs.technet.com/stefan_gossner/archive/2007/11/15/some-comments-on-common-variation-problems.aspx)
> by Stefan Goßner, it appears this will be fixed in a QFE by enabling content
> types on the **Pages** library in destination labels. Excellent!

Here are the repro steps to break the variations feature using no custom code
and no custom content types:

1. Create a new Web application

2. Create a site collection using the **Publishing Portal** site definition

3. Configure variations using / as the **Location** for the **Variation Home**
   (use defaults for all other settings)

4. Create a new variation label with the following:\
   \
   **Label Name: en-US\
   Display Name: English (United States)\
   Locale: English (United States)\
   Source Variation: Yes\
   Publishing site template: Publishing Site with Workflow**

5. Create the variation hierarchies (to create the **/en-US** site)

6. Create a new site under the variation source site (**/en-US/foo**)

7. Change the content type of the default page in the new site
   (**/en-US/foo/default.aspx**) from **Welcome Page** to **Article Page**. Note
   that in order to change the content type of the page, you need to view the
   underlying **Pages** library (use **Site Actions** --&gt; **View All Site
   Content**) and then edit the properties on the page.

8. Change the page layout to **Article page with summary links** and approve the
   page.

9. Create a new variation label with the following:\
   \
   **Label Name: ja-JP\
   Display Name: Japanese\
   Locale: Japanese**

10. Create the variation hierarchies (to create the **/ja-JP** and
    **/ja-JP/foo** sites)

11. View the variation logs and notice the failure with the following error:
    
    {{< blockquote "font-italic text-danger" >}}
    
    The variation system failed to pair up pages
    http://foobar/en-US/foo/Pages/default.aspx and /ja-JP/foo/Pages/default.aspx
    because their Content Types do not match.
    
    {{< /blockquote >}}

[[Part 2](/blog/jjameson/2007/10/31/dumping-moss-2007-variations-part-2) in this
series is now available.]
