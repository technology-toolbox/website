---
title: Creating Wireframes for Page Layouts
date: 2009-06-26T08:34:00-06:00
excerpt:
  When helping customers migrate their Internet sites to Microsoft Office
  SharePoint Server (MOSS) 2007, I've found it very helpful to create wireframes
  showing the various fields, Web Parts, and master page content. For the last
  several years, I've been...
aliases:
  [
    "/blog/jjameson/archive/2009/06/25/creating-wireframes-for-page-layouts.aspx",
    "/blog/jjameson/archive/2009/06/26/creating-wireframes-for-page-layouts.aspx",
  ]
draft: true
categories: ["My System", "SharePoint", "Development"]
tags: ["My System", "MOSS 2007", "Web Development"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/06/26/creating-wireframes-for-page-layouts.aspx"
---

When helping customers migrate their Internet sites to Microsoft Office
SharePoint Server (MOSS) 2007, I've found it very helpful to create wireframes
showing the various fields, Web Parts, and master page content. For the last
several years, I've been doing this in Microsoft Office Visio, but you could
certainly achieve similar results with other tools, such as Expression Design.

I start by taking a series of screenshots of the existing Web site using
[Screengrab!](/blog/jjameson/2008/10/20/fessing-up-about-firefox) (which makes
it very easy to capture entire Web pages). For example, here is a screenshot of
a "Generic" page on the
[Agilent Technologies - LSCA](http://www.chem.agilent.com) site that I captured
a couple of years ago.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/General-Site-Generic-(Glycomics-Solution)-600x477.jpg"
alt="\"Generic\" page" class="screenshot" height="477" width="600"
title="Figure 1: \"Generic\" page" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/General-Site-Generic-%28Glycomics-Solution%29-939x747.jpg)

I then paste the screenshot into Visio.

Next, I deemphasize the portions of the page rendered by the master page. To
accomplish this, add a rectangle with the text **Header (Master Page)** over the
top portion of the page. Then fill the rectangle with light gray (**Shade 15%**)
and set the transparency to **30%**. Repeat similar steps for the footer and
other content provided by the master page.

The next step is to emphasize the various fields that we need to define in the
corresponding content type. In Figure 1, it is easy to discern that we have a
**Title** (i.e. "Glycomics Solution"), a **Page Image** (i.e. the circular image
on the right side of the page), and some arbitrary amount of **Page Content**
(i.e. the "stuff" in the middle). I also chose to provide a **Subtitle** field
for the bold text at the top of the page. Isolating the **Subtitle** allows us
to provide "semantic markup" and better control the formatting and layout of
this content.

To highlight these various fields, I create rectangles with corresponding text
and fill them with a very light red (**Tint 35%**) and again set the
transparency to **30%**.

We now have a "wireframe" that shows the various portions of the page with just
enough visibility of the underlying content to understand what each portion
refers to.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/General-Site-Generic-(Page-Layout)-600x489.jpg"
alt="\"Generic\" page (Page Layout)" class="screenshot" height="489" width="600"
title="Figure 2: \"Generic\" page (Page Layout)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/General-Site-Generic-%28Page-Layout%29-770x627.jpg)

Here is another example, based on a press release.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/General-Site-Press-Release-(857)-478x600.jpg"
alt="Sample Press Release" class="screenshot" height="600" width="478"
title="Figure 3: Sample Press Release" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/General-Site-Press-Release-%28857%29-939x1178.jpg)

Figure 4 shows the corresponding page layout.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/General-Site-Press-Release-(Page-Layout)-461x600.jpg"
alt="Press Release (Page Layout)" class="screenshot" height="600" width="461"
title="Figure 4: Press Release (Page Layout)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/General-Site-Press-Release-%28Page-Layout%29-770x1002.jpg)

Note the importance of choosing good sample pages when mocking up the wireframes
for page layouts. I chose the example press release above based on the fact that
it had "additional contact" information. In other words, all press releases have
primary contact information (which I chose to map to the out-of-the-box
**Contact Name**, **Contact Phone**, and **Contact E-mail Address** fields), but
only some press releases have additional contacts -- which may consist of one
more more individuals. Rather than attempting to create separate fields for each
individual's name, phone number, and e-mail address, we chose to simply provide
a single field for greater flexibility.

Some pages may be comprised of both fields and Web Parts. For example, consider
the following product detail page.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/General-Site-Product-Detail-(6890N-GC)-545x600.jpg"
alt="Sample product detail page" class="screenshot" height="600" width="545"
title="Figure 5: Sample product detail page" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/General-Site-Product-Detail-%286890N-GC%29-940x1034.jpg)

In this scenario, the "Buy Zone" and "Announcements" features are both
implemented as individual Web Parts (due to the dynamic nature of this content).
Figure 6 shows the corresponding page layout, highlighting the Web Parts in a
different color.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/General-Site-Product-Detail-(Page-Layout)-533x600.jpg"
alt="Product Detail (Page Layout)" class="screenshot" height="600" width="533"
title="Figure 7: Product Detail (Page Layout)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/General-Site-Product-Detail-%28Page-Layout%29-770x866.jpg)

Wireframes like these are valuable when you are trying to define the various
content types and page layouts, as well as when it comes time to document your
various feature specs. So the next time you start a SharePoint project -- or any
Web development project for that matter -- I recommend creating a
**Screenshots** library right from the start and adding artifacts like these. I
also keep a copy of the Visio file (typically named Models - Page Layouts.vsd)
so I can quickly make changes as the content types and page layouts evolve over
time.

