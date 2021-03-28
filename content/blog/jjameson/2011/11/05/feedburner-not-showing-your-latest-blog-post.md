---
title: Feedburner not showing your latest blog post? Your feed probably exceeds 512K.
date: 2011-11-05T23:00:01-06:00
excerpt:
  This morning I discovered that Feedburner wasn't showing the blog post I
  created last Thursday. No error was displayed. Rather the RSS feed simply made
  it look like...
aliases:
  [
    "/blog/jjameson/archive/2011/11/05/feedburner-not-showing-your-latest-blog-post.aspx",
  ]
categories: ["Development"]
tags: ["Subtext"]
---

This morning I discovered that Feedburner wasn't showing
[the blog post I created last Thursday](/blog/jjameson/2011/11/03/building-technologytoolbox-com-part-4).

No error was displayed. Rather the RSS feed simply made it look like
"[Part 3](/blog/jjameson/2011/10/27/building-technologytoolbox-com-part-3)" in
my series on building TechnologyToolbox.com was the last post that I created
(having written it myself, I knew that "Part 4" was, in fact, the latest post).

I quickly discovered the following in the "Troubleshootize" page on the
[Google Feedburner](http://feedburner.google.com) site:

> **The trouble:** Your FeedBurner feed isn't up-to-date with your Original
> Feed.\
> **The fix:** Try [pinging FeedBurner](http://feedburner.google.com/fb/a/ping)
> using our Ping page. This action tells FeedBurner to go check your feed for
> updates immediately.

Consequently I "pinged" my blog hoping this would miraculously make my feed show
the latest blog post.

{{< div-block "note" >}}

> **Note**
>
> I had to specify my "Feed Address" (i.e.
> [http://feeds.feedburner.com/Random-Musings-of-Jeremy-Jameson](http://feeds.feedburner.com/Random-Musings-of-Jeremy-Jameson))
> instead of my "Original Feed"
> ([https://www.technologytoolbox.com/blog/jjameson/rss.aspx](/blog/jjameson/rss.aspx))
> when pinging my blog from Feedburner.
>
> According to the **Ping Feedburner** page, you are supposed to provide "the
> address of the Web site hosting the source feed" but I encountered an error
> each time I tried specifying the "Original Feed" address. I'm guessing this is
> caused by the fact that Subtext redirects to the Feedburner URL whenever a
> request is made to "rss.aspx" and the **User-Agent** HTTP header does not
> start with `"FeedBurner"` (refer to the `RedirectToFeedBurnerIfNecessary()`
> method in `BaseSyndicationHandler` if you are using Subtext and you aren't
> sure what I'm talking about here).

{{< /div-block >}}

Unfortunately, after waiting a few minutes, "Part 4" was still nowhere to be
found (except, of course, by browsing directly to my blog).

Shortly thereafter, I discovered the following on the "Troubleshootize" page:

> **The trouble:** Your Original Feed is too doggone big! FeedBurner does not
> process feeds that are larger than **512K**...

To check if my feed exceeded 512K, I created a fresh backup of my Subtext
database in Production, restored it to my Test environment, and subsequently
removed the Feedburner URL specified in the Subtext admin site. [I suppose I
could have just temporarily changed this setting on the PROD site, but I really
don't like "experimenting" on live sites. Also note that I wrote a little SQL
script that makes it very quick and painless to refresh TEST from PROD in just a
couple of minutes.]

When I subsequently browsed to
[http://www-test.technologytoolbox.com/blog/jjameson/rss.aspx](http://www-test.technologytoolbox.com/blog/jjameson/rss.aspx),
I found the blog feed to be 577 KB.

Personally, I'd much rather see Feedburner show an error in this scenario,
rather than making blog subscribers think everything is
"[hunky-dory](http://www.merriam-webster.com/dictionary/hunky-dory)."

The Feedburner site recommends using a query string parameter to reduce the
number of items included in the RSS feed (in order to get it down below 512K).
However, that currently isn't an option with Subtext (since the number of items
included in the RSS feed is tied to the same configuration setting used to
specify the number of items to display on the blog home page).

I briefly considered adding a hack to Subtext to allow a query string parameter
to override the default configuration setting, but then I quickly realized this
scenario shouldn't happen very often. The reason why I experienced this issue
this week is because I included a fairly large amount of HTML sample code in the
two most recent posts on my blog.

Consequently, I decided to simply toggle the option to only include the blog
post description in the RSS feed for "Part 3" and "Part 4" (rather than the
entire content of each post). This definitely isn't how I prefer to syndicate my
blog posts (since it requires people reading my blog to "go online" and download
the full post), but considering the alternatives, this is definitely the logical
choice.
