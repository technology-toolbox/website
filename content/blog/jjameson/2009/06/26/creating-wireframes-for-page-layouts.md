---
title: "Creating Wireframes for Page Layouts"
date: 2009-06-26T02:34:00-07:00
excerpt: "When helping customers migrate their Internet sites to Microsoft Office SharePoint Server (MOSS) 2007, I've found it very helpful to create wireframes showing the various fields, Web Parts, and master page content. For the last several years, I've been..."
draft: true
categories: ["My System", "SharePoint", "Development"]
tags: ["My System", "MOSS 2007", "
                    Web Development"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/06/26/creating-wireframes-for-page-layouts.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/06/26/creating-wireframes-for-page-layouts.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

When helping customers migrate their Internet sites to Microsoft Office SharePoint         Server (MOSS) 2007, I've found it very helpful to create wireframes showing the         various fields, Web Parts, and master page content. For the last several years,         I've been doing this in Microsoft Office Visio, but you could certainly achieve         similar results with other tools, such as Expression Design.

I start by taking a series of screenshots of the existing Web site using [Screengrab!](/blog/jjameson/2008/10/20/fessing-up-about-firefox) (which makes it very easy to capture entire Web pages). For         example, here is a screenshot of a "Generic" page on the [Agilent Technologies - LSCA](http://www.chem.agilent.com) site that I captured a couple of years ago.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www%5Ftechnologytoolbox%5Fcom/blog/jjameson/7/r%5FGeneral%20Site%20-%20Generic%20(Glycomics%20Solution).jpg"
alt="\"Generic\" page"
height="477"    width="600"
title="Figure 1: \"Generic\" page" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_General%20Site%20-%20Generic%20%28Glycomics%20Solution%29.jpg)

I then paste the screenshot into Visio.

Next, I deemphasize the portions of the page rendered by the master page. To accomplish         this, add a rectangle with the text **Header (Master Page)** over the         top portion of the page. Then fill the rectangle with light gray (**Shade 15%**)         and set the transparency to **30%**. Repeat similar steps for the footer         and other content provided by the master page.

The next step is to emphasize the various fields that we need to define in the corresponding         content type. In Figure 1, it is easy to discern that we have a **Title**         (i.e. "Glycomics Solution"), a **Page Image** (i.e. the circular image         on the right side of the page), and some arbitrary amount of **Page Content**         (i.e. the "stuff" in the middle). I also chose to provide a **Subtitle**         field for the bold text at the top of the page. Isolating the **Subtitle**         allows us to provide "semantic markup" and better control the formatting and layout         of this content.

To highlight these various fields, I create rectangles with corresponding text and         fill them with a very light red (**Tint 35%**) and again set the transparency         to **30%**.

We now have a "wireframe" that shows the various portions of the page with just         enough visibility of the underlying content to understand what each portion refers         to.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www%5Ftechnologytoolbox%5Fcom/blog/jjameson/7/r%5FGeneral%20Site%20-%20Generic%20(Page%20Layout).jpg"
alt="\"Generic\" page (Page Layout)"
height="489"    width="600"
title="Figure 2: \"Generic\" page (Page Layout)" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_General%20Site%20-%20Generic%20%28Page%20Layout%29.jpg)

Here is another example, based on a press release.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www%5Ftechnologytoolbox%5Fcom/blog/jjameson/7/r%5FGeneral%20Site%20-%20Press%20Release%20(857).jpg"
alt="Sample Press Release"
height="600"    width="478"
title="Figure 3: Sample Press Release" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_General%20Site%20-%20Press%20Release%20%28857%29.jpg)

Figure 4 shows the corresponding page layout.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www%5Ftechnologytoolbox%5Fcom/blog/jjameson/7/r%5FGeneral%20Site%20-%20Press%20Release%20(Page%20Layout).jpg"
alt="Press Release (Page Layout)"
height="600"    width="461"
title="Figure 4: Press Release (Page Layout)" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_General%20Site%20-%20Press%20Release%20%28Page%20Layout%29.jpg)

Note the importance of choosing good sample pages when mocking up the wireframes         for page layouts. I chose the example press release above based on the fact that         it had "additional contact" information. In other words, all press releases have         primary contact information (which I chose to map to the out-of-the-box **Contact
Name**, **Contact Phone**, and **Contact E-mail Address** fields), but only some press releases have additional contacts -- which         may consist of one more more individuals. Rather than attempting to create separate         fields for each individual's name, phone number, and e-mail address, we chose to         simply provide a single field for greater flexibility.

Some pages may be comprised of both fields and Web Parts. For example, consider         the following product detail page.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www%5Ftechnologytoolbox%5Fcom/blog/jjameson/7/r%5FGeneral%20Site%20-%20Product%20Detail%20(6890N%20GC).jpg"
alt="Sample product detail page"
height="600"    width="545"
title="Figure 5: Sample product detail page" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_General%20Site%20-%20Product%20Detail%20%286890N%20GC%29.jpg)

In this scenario, the "Buy Zone" and "Announcements" features are both implemented         as individual Web Parts (due to the dynamic nature of this content). Figure 6 shows         the corresponding page layout, highlighting the Web Parts in a different color.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www%5Ftechnologytoolbox%5Fcom/blog/jjameson/7/r%5FGeneral%20Site%20-%20Product%20Detail%20(Page%20Layout).jpg"
alt="Product Detail (Page Layout)"
height="600"    width="533"
title="Figure 7: Product Detail (Page Layout)" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_General%20Site%20-%20Product%20Detail%20%28Page%20Layout%29.jpg)

Wireframes like these are valuable when you are trying to define the various content         types and page layouts, as well as when it comes time to document your various feature         specs. So the next time you start a SharePoint project -- or any Web development         project for that matter -- I recommend creating a **Screenshots** library         right from the start and adding artifacts like these. I also keep a copy of the         Visio file (typically named Models - Page Layouts.vsd) so I can quickly make changes         as the content types and page layouts evolve over time.

