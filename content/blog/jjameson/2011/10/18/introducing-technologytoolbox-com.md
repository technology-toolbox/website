---
title: "Introducing TechnologyToolbox.com"
date: 2011-10-18T04:51:49-07:00
excerpt: "In this inaugural post for my new blog location, I'll introduce various features of the new Technology Toolbox website and provide a high-level overview of the underlying architecture. Subsequent posts will cover different aspects of the site in greater detail."
aliases: ["/blog/jjameson/archive/2011/10/17/introducing-technologytoolbox-com.aspx"]
draft: true
categories: ["Development"]
tags: ["Subtext", "Web Development"]
---

In this inaugural post for my new blog location, I'll introduce various features  of the new Technology Toolbox website and provide a high-level overview of the underlying  architecture. Subsequent posts will cover different aspects of the site in greater  detail.

### Home Page

The following screenshot shows the current home page for Technology Toolbox.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Home.png"
alt="Technology Toolbox home page"
height="600"
width="538"
title="Figure 1: Technology Toolbox home page" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Home.png)

As I detailed in a [previous post](/blog/jjameson/2009/06/26/creating-wireframes-for-page-layouts), I find it very helpful to create wireframes that illustrate blocks  of content, navigation elements, and other site features. These often start out  as simple "grey box" diagrams (as described in Andy Clarke's excellent [Transcending CSS](http://www.transcendingcss.com/) book and Jason Santa  Maria's [original
blog post](http://v3.jasonsantamaria.com/archive/2004/05/24/grey_box_method.php)). However, I often update them after the static design is complete  (in other words, after some sample HTML, CSS, and images are sufficiently "baked"  and ready to serve as a reference for the visual design).

Here is the corresponding page layout for the site home page.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Home%20(Page%20Layout).png"
alt="Technology Toolbox home page (page layout)"
height="600"
width="536"
title="Figure 2: Technology Toolbox home page (page layout)" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Home%20%28Page%20Layout%29.png)

#### Branding

The branding area in the masthead is simply comprised of the company logo and  serves as a link back to the site home page from all other pages in the site. Some  sites explicitly include a "Home" link in the main navigation. Personally, these  days I think most people are used to the company logo linking back to the site home  page and thus there's no sense having one more link in the main navigation.

#### Site Search

If your site doesn't provide a way to quickly search for content of interest,  then chances are I am not going to spend much time on your site. I believe this  also holds true for most people who visit TechnologyToolbox.com.

Consequently, I added a ubiquitous "Search" box in the top right corner of the  master page for the site. Currently, the Site Search functionality on TechnologyToolbox.com  is powered by Google. I'll talk more about why Google was chosen for this feature  in a subsequent post and how I integrated Google's custom search functionality into  the site.

#### Main Navigation

Along with branding and site search, the master page for the site also specifies  navigation elements corresponding to the main areas of the site:

- Services
- Company Overview
- Contact Form
- Blog

#### Main Content

The default master page for the site includes a single placeholder for the "main  content" of each page. However, individual content pages (such as the home page)  typically divide this content into two parts: the "primary content" and the "secondary  content." On the home page, the primary content is comprised of a "slider" that  highlights the various services provided by Technology Toolbox (via a series of  rotating images and corresponding captions) and a couple of short paragraphs of  marketing content.

#### Secondary Content

Since the majority of content for the Technology Toolbox site is provided in  the form of blog posts, it seems logical to highlight blog content on the site home  page.

##### Most Recent Posts

As the heading states, the "Most Recent Posts" control simply lists the three  most recent blog posts (in other words, by date published in descending order).  In addition to the post title and date published, this section includes the "summary"  as well as a link to post comments.

##### Most Popular Posts

The "Most Popular Posts" control renders the top ten blog posts based on number  of views. Like the "Most Recent Posts" section, this content is dynamic and easily  generated using the Entity Framework and a little bit of LINQ to query the underlying  database. The content is cached for a short period of time in order to greatly reduce  the database load whenever someone visits the home page.

#### Site Information (a.k.a. Site Footer)

The standard copyright notice appears at the bottom of all pages -- along with  links to the Privacy Policy and Terms of Use.

### Services

The **Services** page for the site is shown in Figure 3 below. This  represents a typical "content" page in which the "primary content" consists of text  with various headings and other markup, while the "secondary content" is comprised  of a related illustration (typically obtained from [iStockphoto](http://www.istockphoto.com) -- my image provider of choice).

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Services.png"
alt="Services page"
height="494"
width="600"
title="Figure 3: Services page" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Services.png)

Note that I still like to use the [960 Grid System](http://960.gs)  to layout Web content -- something I first discussed in a blog post about [Web Standards Design in MOSS 2007](/blog/jjameson/2010/01/30/web-standards-design-with-moss-2007-part-1). The following screenshot shows how the primary  content spans 7 columns and the secondary content spans the remaining 5 columns  (since I chose the 12 column layout option).

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Services-with-Grid.png"
alt="Services page (with grid)"
height="495"
width="600"
title="Figure 4: Services page (with grid)" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Services-with-Grid.png)

In order to minimize the amount of CSS for the site, I chose to extract only  those rules from the 960 Grid System that are currently being used and subsequently  embed those in the "main" CSS file for the TechnologyToolbox.com. This way I don't  have the entire 960.css file linked as a separate style sheet (or the entire file  embedded in my own style sheet). I'll talk more about CSS optimization in a separate  post.

### Company Overview

The "Company Overview" page is very similar to the **Services**  page shown above, so there's really no sense including a screenshot of it in this  post. If you want see what it looks like, just click the **Company** link in the site navigation.

### Contact Form

The site provides an online form prospective clients can use regarding potential  projects and other contact requests, as shown in Figure 5.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Contact.png"
alt="Contact form"
height="600"
width="592"
title="Figure 5: Contact form" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Contact.png)

As shown below, validators are used to ensure the required fields are specified  when submitting the form. Also note how required fields have a light yellow background  color (that changes to white when the field has the focus).

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Contact-validation.png"
alt="Contact form validation errors"
height="600"
width="505"
title="Figure 6: Contact form validation errors" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Contact-validation.png)

To ensure responses are generated by a person -- rather than annoying spam submitted  by some worthless "bot" -- the form includes a CAPTCHA control that requires the  correct image to be selected in order to successfully submit the form. Personally,  I prefer this type of challenge-response (i.e. "image-recognition CAPTCHAs") over  the "distorted text" approach that has become common on many websites today. I often  find it frustrating and time consuming to decipher a random string of characters  that have been mashed together along with a wavy line running through the middle.

When the incorrect image is selected (or no image is selected) in the CAPTCHA  control, an error appears in the validation summary, similar to when required fields  are missing.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Contact-CAPTCHA.png"
alt="Contact form CAPTCHA validation error"
height="600"
width="534"
title="Figure 7: Contact form CAPTCHA validation error" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Contact-CAPTCHA.png)

While the simple image-recognition CAPTCHA is arguably not as "strong" as a distorted  text CAPTCHA (depending, of course, on how much you choose to distort the text),  I think it is sufficient to prevent bots from submitting spam -- at least for now.  I can always make it more robust with a little bit of tweaking if I discover that  someone has written code specifically to hack this particular CAPTCHA implementation.

### Blog

My new blog is currently powered by [Subtext](http://subtextproject.com)  -- or rather my own (slightly modified) version of Subtext 2.5 -- in combination  with a custom blog skin that leverages the same "theme" (i.e. CSS files and associated  images) as the other parts of the site. I discovered some issues with the current  release of Subtext that required some tweaks in order to render the blog pages in  the desired structure and format (for example, to show the **Archives**  section at the bottom of the "secondary content" using expandable lists of links  grouped by year).

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Blog-Home.png"
alt="Blog home page"
height="600"
width="374"
title="Figure 8: Blog home page" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Blog-Home.png)

The primary content on the blog home page is, not surprisingly, the most recent  list of posts (sorted in descending order). The secondary content provides links  for the blog RSS feed, quickly finding blog posts by tags or categories (note that  Subtext supports these concepts differently), and browsing through the historical  archive of posts.

Figure 9 shows a typical blog post. The secondary content is similar to that  shown on the blog home page, but also contains a **Recent Posts** section.  The thought behind this is that people often find blog posts through search sites  like Google or Bing, so when they browse directly to an individual post, we should  try to "hook" them into reading other posts during the same visit. [The **Recent Posts** section is not shown on the blog home page, since this would  be redundant with the items shown in the primary content area on that page.]

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Blog-Post.png"
alt="Sample blog post"
height="600"
width="398"
title="Figure 9: Sample blog post" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Blog-Post.png)

Note that the form for adding a comment to a blog post uses the same custom CAPTCHA  control described previously for the **Contact** form. Getting this  to work as expected was another reason why I had to modify the Subtext solution  for the Technology Toolbox site. Again, I'll cover this in more detail in a subsequent  post.

At first I was little wary of having both "tags" and "categories" for blog posts  (since [my old MSDN blog](http://blogs.msdn.com/b/jjameson) only used  tags). However, after experimenting with a couple of different taxonomies before  settling on my initial list of categories and tags, I discovered that I actually  liked the way Subtext supports these distinct facets.

Other blog-related topics that I plan to cover in subsequent posts include:

- Why I (eventually) chose Subtext as the blog engine for the site after looking
  at numerous other blog solutions
- How I migrated content from my old MSDN blog (running on the Telligent platform)
  to Subtext
- Why I still prefer using Expression Web for authoring blog posts, including
  why I completely disabled the default HTML editor in Subtext

### Logical Architecture

As noted in the previous section, the **Blog** portion of the Technology  Toolbox site is powered by Subtext (an ASP.NET MVC application for which the current  release is built with Visual Studio 2008). However, the other areas of the site  are based on a "classic" ASP.NET Web application built with Visual Studio 2010 (that  targets .NET Framework 3.5).

Originally, I had planned on using .NET Framework 4 for the entire site (and  actually had it running in that configuration for a little while), but that was  before I swapped out the original blog engine for Subtext. [I discovered some serious  scalability issues in the original blog engine selected for the site.]

Upon switching to Subtext, I thought about using ASP.NET MVC across the entire  site. However, I ran into several issues when upgrading the Subtext solution from  Visual Studio 2008 to Visual Studio 2010 -- and to be perfectly honest, I really  don't like working in Visual Studio 2008 these days (unless I absolutely have to).  In order to avoid any further delay, I decided to "punt" and use two different solutions  for the site.

Figure 10 illustrates how the two solutions are merged together during the deployment  process. The **blog** folder is configured as a separate application  in IIS. It contains the Subtext solution and a few updated/additional files from  the "Caelum" solution -- such as the site map file and the custom blog skin.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Solution-Architecture.jpg"
alt="Solution architecture"
height="520"
width="600"
title="Figure 10: Solution architecture" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Solution-Architecture.jpg)

### Physical Architecture

The production environment for TechnologyToolbox.com is currently hosted by [WinHost](http://www.winhost.com), as shown in the following figure.  Separate servers are used for database services (i.e. SQL Server) and the Web tier.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/8/r_Technology-Toolbox-Infrastructure.jpg"
alt="Infrastructure"
height="399"
width="600"
title="Figure 11: Infrastructure" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/8/o_Technology-Toolbox-Infrastructure.jpg)

In addition to Production, separate Development and Test environments are used  to validate releases before deploying to the "live" environment. DEV is automatically  rebuilt at least once a day, whereas TEST is updated "manually" as release candidate  builds become available.

The actual development of the solution is performed on other servers (e.g. FOOBAR7  and FOOBAR2) that are not shown in the above infrastructure model.

