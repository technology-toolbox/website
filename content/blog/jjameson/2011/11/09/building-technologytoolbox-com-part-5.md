---
title: "Semantic HTML for blog pages based on hAtom 0.1 microformat (a.k.a. Building TechnologyToolbox.com, part 5)"
date: 2011-11-09T04:06:22-07:00
lastmod: 2011-11-09T04:07:05-07:00
excerpt: "While creating the new TechnologyToolbox.com site, one of my first tasks was to define the structure of the HTML for the blog pages.

I briefly considered using my old MSDN blog as a reference. However, I quickly dismissed that option after viewing my MSDN blog home page with CSS disabled..."
aliases: ["/blog/jjameson/archive/2011/11/08/building-technologytoolbox-com-part-5.aspx", "/blog/jjameson/archive/2011/11/09/building-technologytoolbox-com-part-5.aspx"]
draft: true
categories: ["Development", "My System"]
tags: ["My System", "Web Development"]
---

While creating the new TechnologyToolbox.com site, one of my first tasks
was to define the structure of the HTML for the blog pages.

I briefly considered using [my
old MSDN blog](http://blogs.msdn.com/b/jjameson/) as a reference. However, I quickly dismissed that option after
viewing my MSDN blog home page with CSS disabled.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/MSDN-blog-home-CSS-disabled-328x600.png"
alt="My MSDN blog home - CSS disabled"
height="600"
width="328"
title="Figure 1: My MSDN blog home - CSS disabled" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/MSDN-blog-home-CSS-disabled-989x1810.png)

One of the most important concepts I learned a few years ago while reading
[Transcending CSS](http://www.transcendingcss.com/) is that your
HTML content should look good "naked" -- in other words, without any CSS rules
applied:

{{< blockquote "font-italic" >}}

...with no distracting layout, the meaningful structure of your naked
content becomes clear: Visitors can more easily see headings and hierarchy,
and they can more easily identify paragraphs, quotations, and lists.

Such meaningful markup and structure simplifies design. Everyone will
benefit from an altogether simpler user experience, one that will be as
easy to navigate on any device from a large monitor to a small-screen mobile
phone. <cite>-- Clarke, Andy. "Semantics Is Meaning."
<a href="http://www.transcendingcss.com/">Transcending CSS</a>. Berkley:
New Riders, 2007: 65.</cite>

{{< /blockquote >}}

Even without clicking the **See full-sized image** link for
Figure 1, it is pretty clear that my MSDN blog does *not* look very good
"naked." Where are the headings? Why do you have to scroll past the list of
tags and the archive list (i.e. posts by month) to see the most recent posts?
More importantly, why does each item in the list of most recent posts start
with a link to "Random Musings of Jeremy Jameson"?!

Compare Figure 1 with the corresponding "naked" version of my new blog home
page:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Technology-Toolbox-Blog-Home-CSS-disabled-362x600.png"
alt="Technology Toolbox blog home page - CSS disabled"
height="600"
width="362"
title="Figure 2: Technology Toolbox blog home page - CSS disabled" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Technology-Toolbox-Blog-Home-CSS-disabled-989x1640.png)

Notice how much easier it is to identify the different sections of the page
-- thanks to the use of HTML heading elements (which were almost nonexistent
on my MSDN blog). Also note that the list of recent posts now appears *before*
the list of tags and the archive list. Each post also includes the metadata
(such as date published and number of comments) rendered as an unordered list
immediately below the post title.

In addition to looking good naked, my new blog conforms to the hAtom 0.1
microformat.

### hAtom 0.1 microformat

Another very useful tip that I picked up from reading Transcending CSS is
using *microformats* as a way of -- as Andy puts it -- <q>"squeezing
new meaning from XHTML."</q>

Consequently, shortly after deciding not to use my MSDN blog as a reference,
I headed over to [http://microformats.org/wiki](http://microformats.org/wiki)
to see if something already existed so I wouldn't have to start from scratch.
That is when I discovered the hAtom 0.1 microformat:

{{< blockquote "font-italic" >}}

hAtom is a microformat for content that can be syndicated, primarily
but not exclusively weblog postings. [...]

...

The hAtom schema consists of the following:

- hfeed (**`hfeed`**). optional.
  - **`feed category`**. optional. keywords or phrases,
    using **[rel-tag](http://microformats.org/wiki/rel-tag "rel-tag")**.
  - hentry (**`hentry`**).
    - **`entry-title`**. required. text.
    - **`entry-content`**. optional (see field description).
      text. [\*]
    - **`entry-summary`**. optional. text.
    - **`updated`**. required using
      [datetime-design-pattern](http://microformats.org/wiki/datetime-design-pattern "datetime-design-pattern"). [\*]
    - **`published`**. optional using
      [datetime-design-pattern](http://microformats.org/wiki/datetime-design-pattern "datetime-design-pattern").
    - **`author`**. required using **[hCard](http://microformats.org/wiki/hcard "hcard")**.
      [\*]
    - **`bookmark`** (permalink). optional, using
      **[rel-bookmark](http://microformats.org/wiki/rel-bookmark "rel-bookmark")**.
    - tags. optional. keywords or phrases, using **[rel-tag](http://microformats.org/wiki/rel-tag "rel-tag")**.

<cite>-- <a href="http://microformats.org/wiki/hatom">http://microformats.org/wiki/hatom</a>
</cite>

{{< /blockquote >}}

Excellent! Now all I had to do was mockup a few sample blog pages in
[my static HTML prototype](/blog/jjameson/2011/10/27/building-technologytoolbox-com-part-3) and subsequently create a custom Subtext skin to
render the same HTML for the live site.

### HTML markup for blog home page

The blog home page (Figure 2) displays a summary of the most recent blog
posts. The entire list is wrapped in an element with `class="hfeed"`
and each post is represented as an element with `class="hentry"`.
The title of each post (a.k.a. "entry") is rendered as an `<h2>`
element with `class="entry-title"`.

```
<div class="hfeed">
  <div class="hentry">
    <h2 class="entry-title">
      <a href="/blog/jjameson/archive/2011/11/06/feedburner-not-showing-your-latest-blog-post.aspx">
        Feedburner not showing your latest blog post? Your feed probably
        exceeds 512K.</a>
    </h2>
    <ul class="post-info">
      <li class="published">
        <span class="label">Published </span>
        <span class="value">November 6, 2011</span>
        <span class="label"> at </span>
        <span class="value">6:00 AM</span>
      </li>
      <li class="vcard author">
        by <span class="fn">Jeremy Jameson</span>
      </li>
      <li class="comments none">
        <a href="/blog/jjameson/archive/2011/11/06/feedburner-not-showing-your-latest-blog-post.aspx#postComments">
          <span class="label">Comments: </span>
          <span class="value count">0</span>
        </a>
      </li>
      <li class="categories">
        <div class="post-categories">
          Categories:
          <ul>
            <li><a rel="tag" href="/blog/jjameson/category/4.aspx">
              Development</a></li>
          </ul>
        </div>
      </li>
    </ul>
    <div class="entry-summary">
      <p>This morning I discovered that Feedburner wasn't showing the blog post
      I created last Thursday. No error was displayed. Rather the RSS feed
      simply made it look like...</p>
    </div>
  </div>
  <div class="hentry">
    <h2 class="entry-title">
      <a href="/blog/jjameson/archive/2011/11/03/building-technologytoolbox-com-part-4.aspx">
      Building TechnologyToolbox.com, Part 4 (a.k.a. Creating a style guide and
      color palette for a Web application)</a>
    </h2>
    <ul class="post-info">
        <li class="published">...</li>
        ...
    </ul>
    <div class="entry-summary">
        <p>...</p>
    </div>
  </div>
  <div class="hentry">
    <h2 class="entry-title">
      <a href="/blog/jjameson/archive/2011/10/27/building-technologytoolbox-com-part-3.aspx">
      Building TechnologyToolbox.com, Part 3 (a.k.a. Creating a static HTML
      prototype for a website)</a>
    </h2>
    <ul class="post-info">
        <li class="published">...</li>
        ...
    </ul>
    <div class="entry-summary">
        <p>...</p>
    </div>
  </div>
  ...
</div>
```

Similar to the [first
example for the hAtom microformat](http://microformats.org/wiki/hatom-examples), I use an unordered list to display the
post metadata (`<ul class="post-info">`). The
corresponding list items then specify the various `class` attributes according to the
hAtom schema (e.g. `<li class="published">`).

> **Note**
>
> At this point, I have deliberately deviated from the
> [datetime-design-pattern](http://microformats.org/wiki/datetime-design-pattern "datetime-design-pattern") for the publication date (i.e. `<li class="published">`)
> due to the known accessibility issues with the Datetime Design Pattern
> (i.e. using an `<abbr>`
> element to represent the date/time with the `title` attribute containing
> the ISO8601 datetime value).
>
> Instead, I chose to use a simpler format based on the
> [Value Class
> Pattern](http://microformats.org/wiki/value-class-pattern).

The post summaries are rendered as paragraphs inside `<div class="entry-summary">`
elements.

By adding the `<span class="label">` elements
to various pieces of text, I can easily hide some portions of the content via
CSS. For example, notice how the first blog post in Figure 2 shows the following:

> Published November 6, 2011 at 6:00 AM

However, when you view the same page with the corresponding CSS enabled,
only the date portion is shown:

> November 6, 2011

Since the list of recent posts will likely contain items that were published
more than 24 hours ago, showing the time version of the publication date seems
superfluous. On the other hand, when viewing an individual blog post, the time
portion *is* shown:

> November 6, 2011     6:00 AM

In both cases, the underlying HTML markup is the same -- it's just that different
CSS rules are applied.

Similarly, I use CSS rules to conditionally show the "comment bubble" icon
along with a link to view the comments for a particular post. For example, in
Figure 3 notice how the **Last Day with Microsoft** post includes
an icon and the corresponding number of comments on the same line as the publication
date, whereas the first post in the list does not. This is accomplished by adding
an additional class to the list item to indicate that a particular post has
no comments (i.e. `<li class="comments none">`).

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Technology-Toolbox-Blog-Home-374x600.png"
alt="Blog home page"
height="600"
width="374"
title="Figure 3: Blog home page" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Technology-Toolbox-Blog-Home-1058x1699.png)

### HTML markup for individual blog post

When viewing a specific blog post, the HTML markup is very similar to the
blog home page. The primary difference is that the `<div class="entry-summary">`
element is replaced by the `<div class="entry-content">`
element.

```
<div id="blogPost">
  <div class="hentry">
    <h2 class="entry-title">
      Feedburner not showing your latest blog post? Your feed probably exceeds
      512K.
    </h2>
    <ul class="post-info">
      <li class="published">
        <span class="label">Published </span>
        <span class="value">November 6, 2011</span>
        <span class="label"> at </span>
        <span class="value">6:00 AM</span>
      </li>
      <li class="vcard author">
        by <span class="fn">Jeremy Jameson</span>
      </li>
      <li class="comments none">
        <a href="#postComments">
          <span class="label">Comments: </span>
          <span class="value count">0</span>
        </a>
      </li>
      <li class="categories">
        <div class="post-categories">
          Categories:
          <ul>
            <li><a rel="tag" href="/blog/jjameson/category/4.aspx">
              Development</a></li>
          </ul>
        </div>
      </li>
    </ul>
    <div class="entry-content">
      <p>This morning I discovered that Feedburner wasn't showing
    <a href="/blog/jjameson/archive/2011/11/03/building-technologytoolbox-com-part-4.aspx.aspx">
      the blog post I created last Thursday</a>.</p>
      <p>No error was displayed. Rather the RSS feed simply made it look like
        "<a href="/blog/jjameson/archive/2011/10/27/building-technologytoolbox-com-part-3.aspx">
          Part 3</a>" in my series on building TechnologyToolbox.com was the
          last post that I created (having written it myself, I knew that
          "Part 4" was, in fact, the latest post).</p>
      ...
  </div>
  ...
</div>
```

The "trick" to formatting the same HTML differently when viewing individual
blog posts (for example, to show the time portion of the publication date) is
to specify a different "container" element than the blog home page (e.g.
`<div id="blogPost">` instead
of `<div id="blogHome">`).

> **Tip**
>
> I like to specify unique "container" elements like this for all pages in a Web application (e.g. `<div  id="companyHome">`). Even though you typically want CSS rules to be very generic (and thus apply to all pages), there may be times when you need to tweak the formatting of some element on a particular page (or set of pages). If each ASP.NET file specifies a unique "container" element, customizing the formatting for specific pages is very easy.

### Resist the urge to change your HTML

Once you have defined the structure of your markup -- and assuming you have
used semantic HTML -- you shouldn't need to change it very much (if at all)
during the process of implementing the desired layout and formatting.

Sure, you may have to do some "tweaking" in order to more easily implement
your design (such as adding a few `id`
and `class` attributes here and
there) -- but, generally speaking, if you've done your job right in defining
your markup, it shouldn't need to be changed every time a client requests a
change to the UI design.

Whenever you are creating a new website -- or adding a new feature to an
existing site -- I encourage you to start by focusing on the "naked" structure.
Once you have defined the meaningful structure for your content, you can then
proceed to making it "look pretty."

