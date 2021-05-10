---
title: '"Git" Your (Infrastructure) Act Together'
date: 2021-04-30T11:27:13-06:00
description:
  Last year I migrated a relatively large OneNote notebook to Markdown -- which
  anyone can access on GitHub. Here is a little background on why this was done
  and how things improved as a result.
categories: ["Infrastructure", "My System"]
tags: ["Infrastructure", "My System"]
---

Last year I migrated a relatively large OneNote notebook to Markdown -- which
anyone can access on GitHub:

{{< reference
title="technology-toolbox / Notebook"
linkHref="https://github.com/technology-toolbox/Notebook" >}}

The notebook has been used for the past 7-1/2 years -- primarily to manage the
infrastructure for Technology Toolbox (regardless of whether that infrastructure
runs on-premises or in Azure).

Prior to that, an "Infrastructure" blog running on SharePoint was used to keep
track of various hardware and software configuration steps, installation
procedures, issues and resolutions, etc. While I liked the chronological nature
of blog posts for tracking infrastructure changes over time, this became a
little cumbersome to maintain. OneNote is simply much faster to interact with
than SharePoint.

OneNote provides features I considered essential -- notably version history and
search -- and a substantially better authoring experience than SharePoint.
(Please don't misunderstand, SharePoint is fantastic for many scenarios, but
quickly creating the kind of formatted content I wanted was not one of them --
even when using wikis and blogs. Also, keep in mind the time period...I'm
talking about SharePoint 2010 and SharePoint 2013 here.)

However, as with all technology, our expectations evolve over time -- to a point
where something that once felt like a huge improvement starts to feel a little
"[long in the tooth](https://www.google.com/search?q=long+in+the+tooth)."

For example, while OneNote automatically saves page versions as you revise
content over time, the version history is limited -- both in terms of when a
page version is saved and how many versions are kept over time. You should think
about OneNote page versions more along the lines of periodically providing a
snapshot that you can reference rather than a traditional version control system
where you can "always diff your changes before commiting."

Similarly, the search feature in OneNote is great for quickly finding a
particular page in a notebook -- but suppose you want to do a simple
["find and replace" in OneNote](https://answers.microsoft.com/en-us/msoffice/forum/msoffice_onenote-mso_win10-mso_o365b/no-find-and-replace-text-seriously/1629f229-0f57-44d2-92e9-46fd1f64cdd5).
For a long time, I was able to "limp along" using
[Onetastic](https://getonetastic.com/), but that falls very short compared to
the other tools I'm used to working with (think Visual Studio and VS Code --
with full support for case matching, regular expressions, etc.).

To modernize my notebook, I decided to leave OneNote behind and actually take
what I consider to be a small step backwards in terms of authoring content --
i.e. using Markdown. It's not that writing Markdown is hard -- on the contrary,
it is an order of magnitude easier than writing similar content in HTML, but it
still feels a little slower than OneNote.

However, what I gained in return for this small step backwards is quite amazing.
For example, every single change I make to the notebook is tracked in Git --
which means I can utilize all the great features of a first-class version
control system. I can now compare the current version of a notebook page with
any previously committed version. I can work on changes in "draft" mode and
subsequently choose to discard or commit my changes (at distinct points in time
that _I choose_ -- not the particular tool I happen to be using).

On the occasions when I need to perform a quick "search and replace" -- for
example when replacing one server with another -- Visual Studio Code makes this
trivial. (If you want to have a small chuckle at Microsoft's expense, you should
check out how -- even as of April 30th, 2021 -- they tell you to
[find and replace text in notes](https://support.microsoft.com/en-us/office/find-and-replace-text-in-notes-34b1f7f8-d327-40c5-8b0c-8419425ed68b).)

There are a few disadvantages of migrating the infrastructure notebook from
OneNote to a public Git repository.

The first is that I have to be very careful about the content added to the
notebook. Anything considered sensitive in nature needs to be redacted or
obscured -- which includes "semi-sensitive" things like hard drive serial
numbers and Azure subscription/tenant IDs.

The second is related to managing images -- especially screenshots. While it is
arguably acceptable to store a small number of images in your Git repo (for
example, "auxillary" images for a website), you should probably avoid storing
hundreds of images in your repo (especially if those images are modified
frequently). GitHub is very generous with their underlying storage, but I recall
reading a number of years ago that they don't want you using GitHub as an "image
hosting service." One option would be to configure Git Large File Storage (LFS)
-- which would at least show that you are willing to "play by the rules" (and
agree to
[GitHub's storage and bandwidth constraints](https://docs.github.com/en/github/managing-large-files/about-storage-and-bandwidth-usage)).

The Technology Toolbox notebook contained over 3,400 images at the time of the
migration from OneNote to GitHub. I chose to store these in Azure Storage and
serve them up through a CDN. Note that
[GitHub proxies the images](https://github.blog/2014-01-28-proxying-user-images/)
when people view them through their website (such that when you view the source
of the pages, it looks like these images are stored in GitHub), but -- trust me
-- all of the images in the Notebook repo are originally served up through
[https://assets.technologytoolbox.com](https://assets.technologytoolbox.com)
(the CDN for the backend Azure Storage resource).

If you are interested in learning more about how this all works "under the
covers" for Technology Toolbox, let me know in the comments below.
