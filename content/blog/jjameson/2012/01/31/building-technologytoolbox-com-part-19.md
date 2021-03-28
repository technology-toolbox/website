---
title: More bug fixes for Subtext now available on GitHub (a.k.a. Building TechnologyToolbox.com, part 19)
date: 2012-01-31T10:10:36-07:00
excerpt:
  In addition to the trivial bug fix that I mentioned in last night's post, I
  have merged a number of other changes into my Subtext fork on GitHub.
aliases:
  [
    "/blog/jjameson/archive/2012/01/31/building-technologytoolbox-com-part-19.aspx",
  ]
categories: ["Development", "My System"]
tags: ["Subtext"]
---

In
[last night's post](/blog/jjameson/2012/01/30/building-technologytoolbox-com-part-18),
I mentioned how I recently created an account on GitHub, primarily so I can
contribute the various bug fixes and enhancements that I've made to the Subtext
blog engine.

In addition to the trivial bug fix that I mentioned before, I have merged a
number of other changes that I previously made to my private build of Subtext.

I'm not sure whether all or any of these changes will find their way into Phil's
repo, but I certainly hope so. Here is a list of the issues that I created (in
most of these, you will see a corresponding change in my private fork of
Subtext):

{{< reference title="Subtext issues created by me"
linkHref="http://github.com/Haacked/Subtext/issues/created_by/jeremy-jameson"
linkText="https://github.com/Haacked/Subtext/issues/created_by/jeremy-jameson" >}}

My Subtext fork does not currently include all of the changes I've made to
Subtext for TechnologyToolbox.com, but it is close.

Here are the changes that I _haven't_ yet merged into my Subtext fork:

- HACK: The current implementation in the `HtmlHelper.ConvertToAllowedHtml`
  method results in extraneous line breaks between paragraphs in comments (e.g.
  "`<p>foo</p>`{newline}\
  `<p>bar</p>`" becomes "`<p>foo</p><br/><p>bar</p>`"). To avoid formatting
  issues, remove the `<br/>` elements added by the `HtmlHelper.HtmlSafe` method
  when paragraphs are separated by newline characters
- Tweak messages displayed to blog readers
- Overhaul the way Gravatar images are handled to provide more flexibility (such
  as showing no image at all if no Gravatar is available -- by specifying
  `PlaceHolderImage=""` in skin Comments.ascx control)
- To avoid numerous `CryptographicException` ("Padding is invalid and cannot be
  removed.") errors from occurring when Google crawls the site, use a static key
  and initialization vector for the CAPTCHA controls
- Change "`Form1`" identifiers to the default ("`aspnetForm`") in order to
  resolve style differences between blog pages and main TechnologyToolbox.com
  site (due to the background gradient being applied to the default
  "`aspnetForm`" element)
- HACK: Don't show "Recent Posts" on the blog home page since this is redundant
  in this scenario
- Change caching strategy in `HomePage` control (in order to avoid issue with
  hiding the "Recent Posts" list)
- HACK: Add an extra `catch` block to avoid a large number of test failures (due
  to expected exceptions not being caught when running unit tests under MbUnit)
- Only change the `Visible` property of the Gravatar image to `true` when an
  email address is specified in the comment. This allows a skin to specify a
  default Gravatar image (e.g.
  `PlaceHolderImage="~/Skins/TechnologyToolbox1/Images/Silhouette-1.jpg"`) but
  also choose to hide the default image when no email address is available to
  retrieve a Gravatar (i.e. by specifying `Visible="false"` on the `Image`
  control)

Honestly, I'm not sure whether these changes would be valuable to others. While
these were all necessary for the TechnologyToolbox.com site, some of them might
be too specific for other sites (especially the ones marked "HACK").

If you have an interest in one or more of the above fixes, let me know and we
can see about getting them merged into my Subtext fork.

Enjoy!
