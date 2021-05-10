---
title: Fessing Up About Firefox
date: 2008-10-20T08:27:00-06:00
description:
  My name is Jeremy, and I'm a Firefox user. There, I've said it. I know, I
  know...what am I thinking, a Microsoft employee telling the world that
  Internet Explorer isn't the end all, be all browser for everyone?! Well, first
  let me clarify a little...
aliases:
  [
    "/blog/jjameson/archive/2008/10/19/fessing-up-about-firefox.aspx",
    "/blog/jjameson/archive/2008/10/20/fessing-up-about-firefox.aspx",
  ]
categories: ["My System", "Development"]
tags: ["My System", "Web Development"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2008/10/20/fessing-up-about-firefox.aspx"
---

My name is Jeremy, and I'm a Firefox user.

There, I've said it.

I know, I know...what am I thinking, a Microsoft employee telling the world that
Internet Explorer isn't the end all, be all browser for everyone?!

Well, first let me clarify a little. I actually do use Internet Explorer as my
primary browser. Internet Explorer 7 is a great browser! Although honestly, I
cringe whenever I think about the large market share still held by IE6. In my
humble opinion, enterprise IT organizations are doing their users a great
disservice by not allowing them -- or, heck, even forcing them -- to upgrade to
IE7. The improved security alone should be enough to justify upgrading to IE7 as
soon as possible.

Now, I know that there were definitely _some_ compatibility issues with IE7
immediately after its release. For example, all of those "bad" Web applications
that were hard-coded to detect version 6, rather than version 6 or later -- but
surely all the owners of those Web applications have resolved these issues by
now ;-)

I also acknowledge that IE6 is actually fairly poor -- at least by comparison --
in its support for [Web standards](http://en.wikipedia.org/wiki/Web_standards).
Anybody who has done any significant amount of Web development can tell you
about their own personal frustrations -- er, I mean _experiences_ -- dealing
with
[box model hacks](http://en.wikipedia.org/wiki/Internet_Explorer_box_model_bug)
and [quirks mode](http://en.wikipedia.org/wiki/Quirks_mode). [I can't tell you
how much I am looking forward to the improved standards support in IE8.]

But alas, the purpose of this post is not to rant about IE6 or IE7, but rather
to tell you about why I often use Firefox.

It is true that Firefox currently supports Web standards, um, _differently_ than
Internet Explorer -- and as a Web developer, I am responsible for making sure
the my pages look great in both Internet Explorer and Firefox (and occasionally
other browsers as well). However, that's not the real reason I use Firefox, or
at least not the most important reason.

Why I use Firefox can easily be summed up in one word: _add-ons_. [Okay,
technically speaking that may actually be two words, but...whatever.]

In the beginning, my dependency on Firefox started with
[Screengrab!](https://addons.mozilla.org/en-US/firefox/addon/1146).

However, over time, and especially after I got serious about Web development
(such as
[dumping table-based layout](http://www.stopdesign.com/articles/throwing_tables/)
and eliminating [spacer GIFs](http://en.wikipedia.org/wiki/Spacer_GIF) in favor
of pure CSS -- &aacute; la, Douglas Bowman), I began to rely heavily on other
add-ons as well -- in particular,
[Web Developer](https://addons.mozilla.org/en-US/firefox/addon/60) and
[Firebug](https://addons.mozilla.org/en-US/firefox/addon/1843).

Most recently, I have also added
[YSlow](https://addons.mozilla.org/en-US/firefox/addon/5369) to my virtual
Toolbox (referring to my standard set of tools that I ensure are installed on
every laptop, desktop, and VM that I develop on).

### My Essential Firefox Add-ons

#### Screengrab!

{{< div-block "border float-end ms-3 p-2" >}}

[![Firefox Add-on - Screengrab!](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Firefox-Screengrab-380x350.png)](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Firefox-Screengrab-801x738.png)

{{< /div-block >}}

When creating feature specs, or other documentation, I often embed screenshots
of Web pages.

Back in the Middle Ages -- or at least before I discovered Screengrab! -- I used
to use {{< kbd "ALT+PRINT SCREEN" >}} to get a an image of the page into
Microsoft Paint, where I would then trim it as necessary and subsequently
copy/paste into a document.

While this works great for certain pages, it is terrribly inefficient when you
want to capture an entire Web page, but the page is too long to fit on screen
(in other words, the vertical scrollbar appears in the browser). Believe it or
not, I actually used to scroll down, {{< kbd "ALT+PRINT SCREEN" >}} again, and
then delicately "append" the new capture to the bottom of the previous capture
in Paint! Needless to say, I didn't do this very often, because it could
literally take a couple of minutes to repeat this process several times for
single page.

Then I discovered a
[utility on CodeProject](http://www.codeproject.com/KB/graphics/IECapture.aspx?fid=192174&df=90&mpp=25&noise=3&sort=Position&view=Quick&fr=101#xx0xx)
that allowed me to capture the entire Web page all at once. This actually worked
fairly well for a while. However, I found that after I upgraded from IE6 to IE7,
I frequently encountered the problem where the captured image would be entirely
black.

Shortly thereafter, I found Screengrab! and I've never since looked for anything
else. It is extremely "lightweight" and, unlike some of the other screen capture
utilities for Firefox, it provides just the essential features:

- Capture only the visible portion of the page, the entire page, or just a
  selected portion
- Copy the image to the clipboard or save directly to a PNG file

The only quirk that I've noticed is that I can't copy a Web page and paste it
directly into Microsoft Office programs. Instead, I still use Paint as an
"intermediary" -- which isn't really an issue due to the speed at which this can
be accomplished with {{< kbd "CTRL+V" >}} (to paste from Screengrab! into
Paint), {{< kbd "CTRL+C" >}} (to copy the entire selection in Paint -- thus
replacing the clipboard content), followed by {{< kbd "CTRL+V" >}} (to paste the
image into Word or Powerpoint).

#### Web Developer

{{< div-block "border float-start me-5 p-2" >}}

[![Firefox Add-on - Web Developer](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Firefox-Web-Developer-380x350.png)](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Firefox-Web-Developer-801x738.png)

{{< /div-block >}}

Chris Pederick's Web Developer add-on provides a toolbar chock-full of goodies
for Web development. Honestly, there are a lot more features in this add-on than
I've ever even begun to use. However, I've found it very helpful simply for the
following:

- Quickly disabling/enabling various browser features, such as caching,
  JavaScript, cookies, and CSS
- Viewing and managing cookies
- Resizing the browser window to view a page in standard sizes, such as 1024x768
  or (shudder) even 800x600
- Outlining specific HTML elements (such as all block level elements) to quickly
  gain familiarity with the structure of a Web page
- Validating HTML, CSS, and JavaScript

A long time ago, I tried out the Internet Explorer Developer Toolbar. However,
while it worked well with IE6, I subsequently discovered issues after I
attempted to install it in IE7. I believe these issues have since been fixed,
but honestly I haven't looked at in a few years.

I am looking forward to the new developer tools in Internet Explorer 8 to see
how they stack up to this add-on and Firebug.

#### Firebug

{{< div-block "border float-end ms-3 p-2" >}}

[![Firefox Add-on - Firebug](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Firefox-Firebug-380x350.png)](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Firefox-Firebug-801x738.png)

{{< /div-block >}}

Compared to anything I had ever seen before or since (including the Web
Developer toolbars for Firefox and Internet Explorer), Firebug takes Web
development to a whole new level.

If you have used the **View Source** feature of your browser more than, say, two
or three times, then you owe it to yourself to stop what you are doing
immediately (that means reading this post), and downloading Firefox and Firebug.

Even if you can do without the previous two add-ons I've covered, if you are a
Web developer I don't see how you can possibly justify continued ignorance to
this beauty of a tool. It is truly amazing!

Seriously, I'm not even going to attempt to tell you how much Firebug has
changed my life -- at least from a Web development perspective.

Get it. Learn it. Use it. Exploit it.

It really is that good! You will quickly start to wonder how you ever managed
without it.

#### YSlow

{{< div-block "border float-start me-3 p-2" >}}

[![Firefox Add-on - YSlow](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Firefox-YSlow-286x380.png)](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Firefox-YSlow-843x1119.png)

{{< /div-block >}}

As I mentioned earlier, I only recently added YSlow to my list of "must have"
add-ons.

Honestly, while I think the "performance report card" that it generates for a
particular page to be helpful, the real reason why I consider this add-on to be
essential is simply the ubiquitous "stopwatch" that it adds to the status bar in
Firefox. With this feature, you no longer have to estimate how long it took to
download a particular page on your site. Rather you can see that, for example,
the current page took 1.032 seconds to download and render.

Personally speaking, I think all browsers should include some kind of
"stopwatch" feature out-of-the-box (or at least the option to turn it on). It is
a fantastic way for team members to quickly answer the question, how is our site
doing in terms of performance? I'm certainly not saying this is a replacement
for load testing and perf monitoring. It's just a good "finger in the air" kind
of thing.

If a Web page is slow, you can then use the other features of YSlow to quickly
investigate the reasons (e.g. large page size, too many external resources that
need to be downloaded, etc.).

Kudos to the Yahoo! development team for packaging up a bunch of their internal
tools and best practices and making them easily usable by Web developers around
the world.

### Conclusion

I want to emphasize again that the intent of this post is not to say that I
prefer Firefox over Internet Explorer, because, honestly, I don't.

To be completely truthful, it's not the Web browser itself that I get excited
about, but rather what gets rendered _inside_ the browser that I occasionally
find myself saying "wow!" when I see it.

I do believe in some ways, we were caught flat footed at Microsoft by Firefox
and are now playing catch-up with Internet Explorer 8. However, it's very
important to keep in mind that competition is a very good thing, especially in
technology. It keeps us innovating and fuels our passion of changing the world
through software.

Will the new developer tools in Internet Explorer 8 be enough to win back some
of the Web developer market share that we have lost? For that, I guess only time
will tell.
