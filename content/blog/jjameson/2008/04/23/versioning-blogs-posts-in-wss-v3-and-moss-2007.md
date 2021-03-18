---
title: Versioning Blog Posts in WSS v3 and MOSS 2007
date: 2008-04-23T10:07:00-06:00
excerpt:
  "Complementing the Work Items list that I described in a previous post , we
  use a blog site (creatively called the \"DevBlog\") in Microsoft Office
  SharePoint Server (MOSS) 2007 to track the work items that each member of the
  Development team has committed..."
aliases:
  [
    "/blog/jjameson/archive/2008/04/22/versioning-blogs-posts-in-wss-v3-and-moss-2007.aspx",
    "/blog/jjameson/archive/2008/04/23/versioning-blogs-posts-in-wss-v3-and-moss-2007.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2008/04/23/versioning-blogs-posts-in-wss-v3-and-moss-2007.aspx](http://blogs.msdn.com/b/jjameson/archive/2008/04/23/versioning-blogs-posts-in-wss-v3-and-moss-2007.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Complementing the **Work Items** list that I described in a
[previous post](/blog/jjameson/2008/04/07/tfs-lite-for-wss-v3), we use a blog
site (creatively called the "DevBlog") in Microsoft Office SharePoint Server
(MOSS) 2007 to track the work items that each member of the Development team has
committed to driving to resolved status each day. For a while, I was the
"keeper" of the DevBlog -- meaning every morning I would gather the targeted
work items for each developer and subsequently add a blog post listing each
developer and his or her committed work items. Fortunately, we've since moved
towards a more autonomous model where each developer is responsible for adding
his or her own work items to the daily blog post.

However, the problem that has been nagging at me is that versioning of the blogs
posts was not working. In other words, if I initially create the blog post for
today (listing the work items that I am committing to resolving) but then
another developer edits the blog post to add his work item commitments, then
when I viewed the version history of the blog post, I noticed that there is only
one version (1.0) even though I had previously changed the list settings for
**Posts** to create a new version each time an item in the list is edited.

In other words, blog post versioning does not work as expected when the
**Posts** list has the following settings:

- **Require content approval for submitted items? - Yes**
- **Create a version each time you edit an item in this list? - Yes**

Ouch.

Mind you, it's not that I don't trust the other developers, I'd just feel a
whole lot better if we could see exactly which changes were made to the work
item commitments blog post by various team members.

After digging into the problem a little, I discovered that versioning works as
expected if you disable content approval for submitted items.

By default, when you create a new site based on the **Blog** template, the
**Posts** list has the following settings:

- **Require content approval for submitted items? - Yes**
- **Create a version each time you edit an item in this list? - No**

It appears that for blog posts, these two settings should be treated as
"mutually exclusive."

If you subsequently change the list settings to the following, then you'll find
that versioning actually works as expected:

- **Require content approval for submitted items? - No**
- **Create a version each time you edit an item in this list? - Yes**

Hopefully this helps others avoid this baffling situation.

Note that the **Blog** template is also available in Windows SharePoint Services
(WSS) v3.
