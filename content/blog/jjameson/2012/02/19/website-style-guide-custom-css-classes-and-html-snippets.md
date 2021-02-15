---
title: "Website style guide, custom CSS classes, and HTML snippets"
date: 2012-02-19T02:17:03-07:00
excerpt: "If, like me, you use Expression Web to author HTML content, then I hope you take advantage of the \"Snippets\" feature."
draft: true
categories: ["My System"]
tags: ["My System", "Subtext", "Web Development"]
---

Time to get rid of another couple of
[shortcuts on my Windows desktop](/blog/jjameson/2012/02/19/stop-putting-shortcuts-on-my-windows-desktop) that have been parked there far too long:

- **MDC style guide - MDC**
  [http://developer.mozilla.org/Project:En/MDC\_style\_guide](http://developer.mozilla.org/Project:En/MDC_style_guide)
- **Custom CSS Classes - MDC**
  [http://developer.mozilla.org/Project:en/Custom\_CSS\_Classes](http://developer.mozilla.org/Project:en/Custom_CSS_Classes)

When I stumbled upon these pages a few years ago, I immediately thought of
that quote from the [old
Guinness commercials](http://www.youtube.com/watch?v=3DPKf7y1F-Q):

> Brilliant!

I doubt the Mozilla folks were the first to fomalize their style guide and
CSS rules into a "document" like this, but kudos to them anyway for sharing
their work with the rest of us.

It inspired me to start creating style guides for websites and Web applications
that I work on. You can see an example in
[one of my previous posts](/blog/jjameson/2011/11/03/building-technologytoolbox-com-part-4). In retrospect, I should have referenced these
sources in that post but, well, I somehow forgot. Lo siento.

In the Template.aspx page that I use for creating new blog posts, I include
sample HTML blocks that demonstrate the custom CSS classes used throughout my
blog posts. These HTML blocks are very similar to those shown in the style guide
from my earlier post, so no sense repeating them here.

To quickly insert commonly used HTML fragments into blog posts (including
custom CSS classes), I created some custom snippets in Expression Web. Yes,
[I still use Expression Web for authoring blog posts](/blog/jjameson/2009/09/12/expression-web-my-msdn-blog-and-now-team-foundation-server) rather than using the
rich HTML editor in Subtext.

One reason for this is that I'd much rather have a
[WYSIWYG](http://en.wikipedia.org/wiki/Wysiwyg) experience when writing
content for the Web. I also discovered that the version of the HTML editor in
Subtext 2.5 royally screws up preformatted content (i.e. `<pre>`
elements) -- which my blog tends to have a lot of. Consequently I ended up disabling
that feature entirely. Note that this is not
[Phil](http://www.haacked.com)'s fault -- he simply used one of the
freely available rich HTML editors. Oh, and if it makes you feel any better,
I found that Visual Studio 2010 also screws up `<pre>` content
:-(

I find these snippets to be faster than copying/pasting HTML from the content
included in the Template.aspx file.

Here is a screenshot that illustrates the snippets I've created.

![Snippets in Expression Web](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Expression%20Web%20-%20My%20Technology%20Toolbox%20Blog.png)

    	Figure 1: Snippets in Expression Web

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Expression%20Web%20-%20My%20Technology%20Toolbox%20Blog.png)
