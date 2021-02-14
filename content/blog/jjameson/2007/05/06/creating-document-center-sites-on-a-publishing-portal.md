---
title: "Creating Document Center Sites on a Publishing Portal"
date: 2007-05-06T02:34:00+08:00
excerpt: "Here is another issue I discovered long ago, but haven't blogged about yet. Shame on me. 
 When kicking off my current project back in December, I was convinced that a Document Center site in Microsoft Office SharePoint Server (MOSS) 2007 was the best..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
> 
>             This post originally appeared on my MSDN blog:
> 
> 
> 
> [http://blogs.msdn.com/b/jjameson/archive/2007/05/06/creating-document-center-sites-on-a-publishing-portal.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/05/06/creating-document-center-sites-on-a-publishing-portal.aspx)
> 
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.


Here is another issue I discovered long ago, but haven't blogged about yet. Shame         on me.

When kicking off my current project back in December, I was convinced that a Document         Center site in Microsoft Office SharePoint Server (MOSS) 2007 was the best choice         for migrating my customer's legacy document management system. My primary reasons         were the enhanced navigation and other features for supporting an "enterprise document         library" with a large number of documents. However, knowing that this is an Internet-facing         site, I also wanted to use the Publishing Portal template for the main site collection         (for rich Web content management features and content approval workflows).

Here is the excerpt from the email I sent back in December to one of our internal         DLs:


> Even if you enable the Document Center template on a "Publishing with Workflow"             site, it still does not show up when you attempt to create a subsite.
> 
> What is really strange is that I when also enable the Records Center template, then             I have the option to create a Records Center when creating a subsite. So there either             must be some restrictions on where a Document Center can be used or else this is             a bug -- and a nasty bug at that, considering a single document repository is one             of the core features that my customer needs for their externally facing (i.e. Publishing             with Workflow) site.


It turns out that it is a bug, but a very minor bug and one that is easy to workaround.         After a little experimentation, I discovered that the **Page Layout and Site Template
            Settings** page (/\_layouts/AreaTemplateSettings.aspx) ignores any feature         dependencies of the site templates (and therefore shows you all available tempates)         whereas the **New SharePoint Site** page (/\_layouts/newsbweb.aspx)         apparently does not. Therefore if you don't have the features required by the Document         Center site enabled on the collection, then you cannot create a new Document Center         site.

The workaround is to enable *both* the Enterprise and Standard features on         the site collection. Then you can add a Document Center to a Publishing Portal site         collection.

