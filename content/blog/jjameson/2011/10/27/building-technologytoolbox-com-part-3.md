---
title: "Creating a static HTML prototype for a website (a.k.a. Building TechnologyToolbox.com, part 3)"
date: 2011-10-27T10:59:07-06:00
excerpt:
  "Regardless of the platform a website will eventually run on (e.g. ASP.NET or
  SharePoint), I typically recommend creating a static HTML prototype to
  demonstrate key features, illustrate various design alternatives (e.g.
  different page layouts, color schemes, etc.), and gather feedback..."
aliases: ["/blog/jjameson/archive/2011/10/27/building-technologytoolbox-com-part-3.aspx"]
draft: true
categories: ["Development", "My System"]
tags: ["Web Development"]
---

Regardless of the platform a website will eventually run on (e.g. ASP.NET or
SharePoint), I typically recommend creating a static HTML prototype to
demonstrate key features, illustrate various design alternatives (e.g. different
page layouts, color schemes, etc.), and gather feedback from Product Management.

Using a tool like Expression Web to create a mockup of site pages can
substantially reduce the amount of time spent developing ASP.NET and SharePoint
applications.

### What does "static" really mean?

The important thing to realize about what I call the "static HTML prototype" is
that while it may include things like JavaScript and ASP.NET master pages, it
shouldn't have any dependencies on external data (i.e. all of the content
displayed in the prototype is "static" or constant) and it shouldn't support any
"real" functionality like logging in to the site. You can definitely mock up a
login form -- if this is an important feature that you want to gather feedback
on -- but the prototype shouldn't actually authenticate a user when someone
fills out the login form.

Whether you choose to leverage ASP.NET master pages in the HTML prototype is
essentially a tradeoff between how easy you want it to be to "run" your
prototype vs. how easy you want to be able to change the common HTML markup
shared across multiple pages (e.g. the masthead and global navigation).

I've been known to zip up static HTML prototypes and embed the compressed file
in a Microsoft Word document (or attach the file to an email message) and send
to key stakeholders who subsequently unzip and view the prototype on their own
laptops. Obviously if you are using ASP.NET master pages in your HTML prototype
(rather than plain HTML files), this signficantly changes the prerequisites for
running the prototype. Depending on the target audience -- and how you choose to
have them access the prototype -- using ASP.NET master pages may or may not be a
viable option.

### Introducing the "Caelum" prototype

One of the first things I needed to do for the new TechnologyToolbox.com site
was design the various blog pages (e.g. the blog home page, a typical summary
page with all posts for a particular tag, the layout for an individual post,
etc.).

Using Expression Web, I created a site under the TFS workspace for the Caelum
project (**$/Caelum/Dev/CaelumPrototype**). I then added various folders and
files corresponding to the basic structure of the website.

{{< table class="small"
caption="Sample HTML content for the Caelum prototype" >}}

| File | Description |
| --- | --- |
| Default.master | Default master page for the site |
| Default.aspx | Site home page |
| blog\jjameson\BlogPost.master | Master page used for viewing individual blog posts |
| blog\jjameson\Default.aspx | Blog home page |
| blog\jjameson\archive\2011\08\22\leaving-microsoft.aspx | Sample post |
| blog\jjameson\archive\2011\09\02\last-day-with-microsoft.aspx | Sample post |
| blog\jjameson\archive\2011\09\02\new-blog-location.aspx | Sample post |

{{< /table >}}

### Default.master

Here is the current content of the default master page for the static HTML
prototype:

```
<%@ Master Language="C#" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">

<head>
  <meta content="text/html; charset=utf-8" http-equiv="Content-Type" />
  <title>Technology Toolbox</title>
  <link href="/Themes/Theme-1.0/Main.css" rel="stylesheet" type="text/css" />
  <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.4.2/jquery.js" type="text/javascript"></script>
  <asp:ContentPlaceHolder id="AdditionalHeadContent" runat="server" />
</head>

<body id="technology-toolbox-com">
<form id="aspnetForm" runat="server">
  <div id="wrapper">
    <div id="branding">
      <h1><a href="/">
      <img alt="Technology Toolbox" src="/Images/Technology-Toolbox-Logo-220x50.png" /></a></h1>
      <blockquote>
        <p>Your technology Sherpa for the Microsoft platform</p>
        <p><cite>Jeremy Jameson - Founder and Principal</cite></p>
      </blockquote>
    </div>
    <div id="siteSearch">
      <h2>Search</h2>
      <input type="text" id="searchKeywords" value="Search..."/>
      <a href="javascript:submitSearch()"><img alt="Search" src="Images/icon-search-22x22.png" /></a>
      <script type="text/javascript">
        function submitSearch() {
          var $searchKeywords = $('#searchKeywords');

          var keywords = $searchKeywords.val();

          window.location.href = "/Search.aspx?q=" + encodeURIComponent(keywords);
        }
      </script>
    </div>
    <div id="navMain">
      <h2>Site features</h2>
      <ul id="navFeatures">
        <li><a href="/Services">Services</a></li>
        <li><a href="/Company">Company</a></li>
        <li><a href="/Contact">Contact</a></li>
        <li><a href="/blog/jjameson">Blog</a></li>
      </ul>
    </div>
    <div id="pageContentPlaceholder" class="clear-fix">
      <asp:ContentPlaceHolder id="MainContent" runat="server" />
    </div>
    <div id="siteInfo" class="clear-fix">
      <hr />
      <h2 class="link-to-top">
      <a href="#technology-toolbox-com" title="Back to top">Technology Toolbox</a></h2>
      <ul>
        <li>Copyright &copy; 2011 Technology Toolbox - All rights reserved.</li>
        <li><a href="/Terms-of-Use.aspx">Terms of Use</a></li>
        <li><a href="/Privacy-Policy.aspx">Privacy Policy</a></li>
      </ul>
    </div>
  </div>
</form>
</body>

</html>
```

> **Note**
>
> Rather than trying to understand all of the discrete HTML elements shown in
> this post, focus instead on the high-level structure of the various pages and
> how they fit together. I'll discuss the semantic HTML used for the Technology
> Toolbox pages in more detail in subsequent posts.

Figure 1 shows the corresponding Design view of the master page in Expression
Web (after adding the CSS rules to define the site fonts, colors, etc.).

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Caelum-Default.master-600x142.png"
alt="Caelum prototype - Default.master" class="screenshot" height="142"
width="600" title="Figure 1: Caelum prototype - Default.master" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Caelum-Default.master-1021x241.png)

### Default.aspx

The site home page references Default.master and specifies its content using the
**MainContent** placeholder as well as the additional CSS and JavaScript files
needed to support the "slideshow" content (via the **AdditionalHeadContent**
placeholder):

```
<%@ Page Title="" Language="C#" MasterPageFile="~/Default.master" %>
<asp:Content runat="server" ContentPlaceHolderID="AdditionalHeadContent">
  <script type="text/javascript" src='/Scripts/jquery.nivo.slider.js'></script>
    <link rel="stylesheet" href="/Themes/Nivo-Slider/nivo-slider.css" type="text/css" media="screen" />
</asp:Content>
<asp:Content runat="server" ContentPlaceHolderID="MainContent">
<div id="siteHome">
  <div id="content" class="container_12">
    <div id="contentMain" class="grid_12">
      <div class="slider-wrapper">
        <div class="ribbon"></div>
        <div id="slider" class="nivoSlider"style="width:940px;height:380px;">
          <img alt="" height="380" src="Images/Slider-SharePoint-Architecture-and-Development-940x380.jpg" title="#sharepointCaption" width="940" />
          <img alt="" height="380" src="Images/Slider-Branching-Strategy-940x380.jpg" title="#almCaption" width="940" />
          <img alt="" height="380" src="Images/Slider-SQL-Server-Business-Intelligence-940x380.jpg" title="#biCaption" width="940" />
        </div>
        <div id="sharepointCaption" class="nivo-html-caption">
          <h2>
          <a href="/Services/SharePoint-Architecture-and-Development.aspx">
          SharePoint Architecture and Development</a></h2>
          <p>Are you upgrading or migrating to SharePoint 2010? We have
          the experience and best practices to ensure your success.</p>
        </div>
        <div id="almCaption" class="nivo-html-caption">
          <h2>
          <a href="/Services/Application-Lifecycle-Management.aspx">Application
          Lifecycle Management</a></h2>
          <p>Whether you are building on SharePoint or the core .NET Framework,
          you need a solid foundation for continuously managing the development
          lifecycle.</p>
        </div>
        <div id="biCaption" class="nivo-html-caption">
          <h2>
          <a href="/Services/Databases-and-Business-Intelligence.aspx">
          Databases and Business Intelligence</a></h2>
          <p>Need help troubleshooting SQL Server or empowering business
          users with sophisticated analytics solutions? We can help with
          that, too.</p>
        </div>
      </div>
      <script type="text/javascript">
        $(window).load(function () {
          $('#slider').nivoSlider({
            effect: "fade",
            pauseTime: 5000
          });
        });
      </script>
      <h2>Your technology Sherpa for the Microsoft platform</h2>
      <p>The path to success in the world of information technology is neither
      flat nor easy. It is often steep, rugged, and filled with hazards and
      pitfalls. However, partnering with an experienced guide vastly reduces
      the risk, effort, and duration of the journey.</p>
      <p>Technology Toolbox helps businesses thrive by maximizing their investments
      in Microsoft products and technologies, including SharePoint, Team Foundation
      Server, and SQL Server.</p>
    </div>
    <div id="contentSub">
      <div class="grid_6">
        <div class="hfeed posts-recent">
          <h2>Most Recent Posts</h2>
          <div class="hentry">
            <h3 class="entry-title">
            <a href="/blog/jjameson/archive/2011/09/02/last-day-with-microsoft.aspx" rel="bookmark">
            Last Day with Microsoft</a></h3>
            <ul class="post-info">
              <li class="published"><span class="label">Published
              </span><span class="value">September 2, 2011</span><span class="label">
              at </span><span class="value">3:43 PM</span></li>
              <li class="vcard author">by <span class="fn">Jeremy
              Jameson</span></li>
              <li class="comments">
              <a href="/blog/jjameson/archive/2011/09/02/last-day-with-microsoft.aspx#postComments">
              <span class="label">Comments: </span>
              <span class="value count">11</span></a></li>
              <li class="categories">
              <div class="post-categories">
                Categories:
                <ul>
                  <li>
                  <a href="/blog/jjameson/category/1.aspx" rel="tag" title="View all posts in this category">
                  Personal</a></li>
                </ul>
              </div>
              </li>
            </ul>
            <div class="entry-summary">
              <p>Today is my last day with Microsoft and it is with
              a heavy heart that I now say farewell to past and present
              teammates, coworkers, and managers...</p>
            </div>
          </div>
          <div class="hentry">
            <h3 class="entry-title">
            <a href="/blog/jjameson/archive/2011/09/02/new-blog-location.aspx" rel="bookmark">
            New Blog Location - https://www.technologytoolbox.com/blog/jjameson</a></h3>
            <ul class="post-info">
              <li class="published"><span class="label">Published
              </span><span class="value">September 2, 2011</span><span class="label">
              at </span><span class="value">1:59 PM</span></li>
              <li class="vcard author">by <span class="fn">Jeremy
              Jameson</span></li>
              <li class="comments none">
              <a href="/blog/jjameson/archive/2011/09/02/new-blog-location.aspx#postComments">
              <span class="label">Comments: </span>
              <span class="value count">0</span></a></li>
              <li class="categories">
              <div class="post-categories">
                Categories:
                <ul>
                  <li>
                  <a href="/blog/jjameson/category/1.aspx" rel="tag" title="View all posts in this category">
                  Personal</a></li>
                </ul>
              </div>
              </li>
            </ul>
            <div class="entry-summary">
              <p>As I mentioned in my previous post , today is my
              last day with Microsoft. I still have a lot of work
              to do in setting up my new website, but at this point,
              I've completed enough to continue blogging the "Random
              Musings of Jeremy Jameson": http:...</p>
            </div>
          </div>
          <div class="hentry">
            <h3 class="entry-title">
            <a href="/blog/jjameson/archive/2011/08/22/leaving-microsoft.aspx" rel="bookmark">
            Leaving Microsoft</a></h3>
            <ul class="post-info">
              <li class="published"><span class="label">Published
              </span><span class="value">August 22, 2011</span><span class="label">
              at </span><span class="value">7:03 AM</span></li>
              <li class="vcard author">by <span class="fn">Jeremy
              Jameson</span></li>
              <li class="comments none">
              <a href="/blog/jjameson/archive/2011/08/22/leaving-microsoft.aspx#postComments">
              <span class="label">Comments: </span>
              <span class="value count">0</span></a></li>
              <li class="categories">
              <div class="post-categories">
                Categories:
                <ul>
                  <li>
                  <a href="/blog/jjameson/category/1.aspx" rel="tag" title="View all posts in this category">
                  Personal</a></li>
                </ul>
              </div>
              </li>
            </ul>
            <div class="entry-summary">
              <p>Last Thursday, I informed my manager that I have
              decided to leave Microsoft to pursue other opportunities.
              My last day will be September 2nd -- just a few days
              shy of my 11 year anniversary date with the company
              (my first day was September 5, 2000...</p>
            </div>
          </div>
        </div>
      </div>
      <div class="grid_6">
        <div class="posts-most-popular">
          <h2>Most Popular Posts</h2>
          <ol>
            <li><a href="#">Issues Deploying SharePoint Solution Packages</a></li>
            <li><a href="#">The Case of the Disappearing Hosts File</a></li>
            <li><a href="#">Upgrade Team Foundation Server 2008 to TFS
            2010 (and SharePoint Server 2010)</a></li>
            <li><a href="#">Dumping MOSS 2007 Variations - Part 1</a></li>
            <li><a href="#">The workbook cannot be opened Error with
            SharePoint Server 2010 (and TFS 2010)</a></li>
            <li><a href="#">Error Creating Control when using Microsoft
            Office SharePoint Designer 2007</a></li>
            <li><a href="#">Best Practices for .NET Assembly Versioning</a></li>
            <li><a href="#">Counting Rows in All Database Tables in
            SQL Server</a></li>
            <li><a href="#">Creating a Site Template in MOSS 2007 that
            Works in WSS v3</a></li>
            <li><a href="#">Outlook 2010 Does Not Work with Windows
            Server 2003 POP3 Service</a></li>
          </ol>
        </div>
      </div>
    </div>
  </div>
</div>
</asp:Content>
```

The following screenshot shows the home page rendered in a browser. [This
screenshot is actually from the live site -- not the static HTML prototype.
However, the only difference with the prototype version is the first item under
the **Most Recent Posts** section. Hence I didn't bother capturing a different
screenshot from the HTML prototype.]

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Technology-Toolbox-Home-538x600.png"
alt="Technology Toolbox home page" class="screenshot" height="600" width="538"
title="Figure 2: Technology Toolbox home page" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Technology-Toolbox-Home-1058x1179.png)

### BlogPost.master

When viewing an individual blog post, a different master page is used in order
to render common elements such as the sidebar content (e.g. **Recent Posts** and
**Tags**) as well as the comment form.

BlogPost.master defines a new **ContentPlaceHolder** control (i.e.
**PostContent**) that is subsequently used to specify the content of a specific
blog post.

```
<%@ Master Language="C#" MasterPageFile="~/Default.master"%>
<asp:Content runat="server" ContentPlaceholderID="MainContent">
<div id="blogPost">
  <div id="content" class="container_12">
    <h1 class="blog-title">Random Musings of Jeremy Jameson</h1>
    <div id="contentMain" class="grid_8">
      <asp:ContentPlaceHolder id="PostContent" runat="server" />
      <div id="commentsUpdatePanelWrapper">
        <div id="postComments">
          <h3>Comments</h3>
          <a href="#respond">Add a comment</a>
          <asp:ContentPlaceHolder runat="server" id="PostComments">No comments posted yet.</asp:ContentPlaceHolder>
        </div>
        <div id="commentForm">
          <h3>Add Comment</h3>
          <fieldset>
          <div>
            <label for="ctl11_tbName">Name</label>
            <input id="ctl11_tbName" class="required" maxlength="32" name="ctl11$tbName" size="20" type="text" />
            <span id="ctl11_ctl01" class="validator required" style="display: inline;">
            (required)</span> </div>
          <div>
            <label for="ctl11_tbTitle">Title</label>
            <input id="ctl11_tbTitle" class="required" maxlength="128" name="ctl11$tbTitle" size="20" type="text" value="re: PowerShell Script to Configure Search in SharePoint Server 2010" />
            <span id="ctl11_ctl03" class="validator required" style="display: inline;">
            (required)</span> </div>
          <div>
            <label for="ctl11_tbEmail">Email Address</label>
            <input id="ctl11_tbEmail" maxlength="128" name="ctl11$tbEmail" size="20" type="text" value="foobar" />
            <span id="ctl11_ctl05" class="validator" style="display: inline;">
            (invalid)</span>
            <div class="field-info">
              Optional, but recommended (especially if you have a
              <a href="http://www.gravatar.com" target="_blank">Gravatar</a>).
              Note that your email address will not appear with your
              comment.</div>
          </div>
          <div>
            <label for="ctl11_tbUrl">URL</label>
            <input id="ctl11_tbUrl" maxlength="256" name="ctl11$tbUrl" size="20" type="text" />
            <span id="ctl11_ctl07" class="validator" style="display: none;">
            (invalid)</span>
            <div class="field-info">
              If URL is specified, it will be included as a link with
              your name.</div>
          </div>
          <div>
            <span class="checkbox">
            <input id="ctl11_chkRemember" checked="checked" name="ctl11$chkRemember" type="checkbox" /><label for="ctl11_chkRemember">Remember
            me</label></span> </div>
          <div>
            <label for="ctl11_tbComment">Comment</label>
            <textarea id="ctl11_tbComment" class="required" cols="60" name="ctl11$tbComment" rows="10"></textarea>
            <span id="ctl11_ctl09" class="validator required" style="display: inline;">
            (required)</span></div>
          </fieldset>
          <div class="button-panel">
            <span style="color: red; display: none;">Please enter the
            answer to the supplied question (custom).</span><span style="display: none;">Please
            add 2 and 1 and type the answer here:
            <input name="ctl11_ctl11_visibleanswer" type="text" value="" /></span><input name="ctl11_captcha_encrypted" type="hidden" value="D3w3iOYrarnJ9qs3gAEHj/nCD+z74YvsRCUaxHXKbbU=" /><div class="captcha">
              <img border="0" height="50" src="/blog/images/services/CaptchaImage.ashx?spec=mbDRk9FMNIIe8ZXgEkkr8nOdGg4Opaxs5Bex2UB6s3AyO5RQSj8RNb62WWhANl%2bC" width="180" /><label for="ctl11_captcha_answer">Enter
              the code shown above:</label><span style="width: 180px; height: 50px; color: red; display: none;">Please
              enter the correct word</span><input maxlength="4" name="ctl11_captcha_answer" size="4" type="text" value="" /></div>
            <input id="ctl11_btnSubmit" name="ctl11$btnSubmit" onclick="javascript:WebForm_DoPostBackWithOptions(new WebForm_PostBackOptions(&quot;ctl11$btnSubmit&quot;, &quot;&quot;, true, &quot;SubtextComment&quot;, &quot;&quot;, false, false))" type="submit" value="Submit" />
          </div>
          <div id="ctl11_ctl10" class="validation-summary" style="">
            There is a problem with your request. Please correct and
            try again.<ul>
              <li>Name must be specified.</li>
              <li>Title must be specified.</li>
              <li>Email Address is not required, but if specified
              it must be valid.</li>
              <li>Comment must be specified.</li>
            </ul>
          </div>
        </div>
      </div>
    </div>
    <div id="contentSub" class="grid_4">
      <p class="rss-feed">
        <a href="http://feeds.feedburner.com/Random-Musings-of-Jeremy-Jameson">
        <img alt="RSS icon" height="30" src="/Images/icon-rss-30x30.png" width="30" />
        Subscribe to this blog (RSS)</a>
      </p>
      <div class="posts-recent">
        <h2>Recent posts</h2>
        <ul>
          <li>
          <a href="/blog/jjameson/archive/2011/09/02/last-day-with-microsoft.aspx">
          Last Day with Microsoft</a></li>
          <li>
          <a href="/blog/jjameson/archive/2011/09/02/new-blog-location-http-www-technologytoolbox-com-blog-jjameson.aspx">
          New Blog Location - https://www.technologytoolbox.com/blog/jjameson</a></li>
          <li>
          <a href="/blog/jjameson/archive/2011/08/22/leaving-microsoft.aspx">
          Leaving Microsoft</a></li>
          <li>
          <a href="/blog/jjameson/archive/2011/05/05/using-the-sharepoint-api-to-configure-an-expiration-policy-on-a-document-library.aspx">
          Using the SharePoint API to Configure an Expiration Policy on
          a Document Library</a></li>
          <li>
          <a href="/blog/jjameson/archive/2011/05/02/missing-thumbnail-images-in-sharepoint-you-probably-forgot-to-specify-the-quot-contenttype-quot-property.aspx">
          Missing thumbnail images in SharePoint?...You probably forgot
          to specify the "ContentType" property</a></li>
        </ul>
      </div>
      <div class="posts-tags">
        <h2>Tags</h2>
        <ul>
          <li class="tag-weight-3">
          <a href="/blog/jjameson/tags/Core%20Development/default.aspx" title="Core Development (55)">
          Core Development</a></li>
          <li class="tag-weight-2">
          <a href="/blog/jjameson/tags/Debugging/default.aspx" title="Debugging (7)">
          Debugging</a></li>
          <li class="tag-weight-3">
          <a href="/blog/jjameson/tags/Infrastructure/default.aspx" title="Infrastructure (45)">
          Infrastructure</a></li>
          <li class="tag-weight-3">
          <a href="/blog/jjameson/tags/MOSS%202007/default.aspx" title="MOSS 2007 (130)">
          MOSS 2007</a></li>
          <li class="tag-weight-3">
          <a href="/blog/jjameson/tags/My%20System/default.aspx" title="My System (84)">
          My System</a></li>
          <li class="tag-weight-2">
          <a href="/blog/jjameson/tags/Personal/default.aspx" title="Personal (13)">
          Personal</a></li>
          <li class="tag-weight-2">
          <a href="/blog/jjameson/tags/PowerShell/default.aspx" title="PowerShell (14)">
          PowerShell</a></li>
          <li class="tag-weight-3">
          <a href="/blog/jjameson/tags/SharePoint%20Server%202010/default.aspx" title="SharePoint Server 2010 (47)">
          SharePoint Server 2010</a></li>
          <li class="tag-weight-2">
          <a href="/blog/jjameson/tags/Simplify/default.aspx" title="Simplify (22)">
          Simplify</a></li>
          <li class="tag-weight-2">
          <a href="/blog/jjameson/tags/SQL%20Server/default.aspx" title="SQL Server (15)">
          SQL Server</a></li>
          <li class="tag-weight-3">
          <a href="/blog/jjameson/tags/TFS/default.aspx" title="TFS (40)">
          TFS</a></li>
          <li class="tag-weight-2">
          <a href="/blog/jjameson/tags/Toolbox/default.aspx" title="Toolbox (26)">
          Toolbox</a></li>
          <li class="tag-weight-2">
          <a href="/blog/jjameson/tags/Virtualization/default.aspx" title="Virtualization (22)">
          Virtualization</a></li>
          <li class="tag-weight-2">
          <a href="/blog/jjameson/tags/Visual%20Studio/default.aspx" title="Visual Studio (27)">
          Visual Studio</a></li>
          <li class="tag-weight-2">
          <a href="/blog/jjameson/tags/Web%20Development/default.aspx" title="Web Development (24)">
          Web Development</a></li>
          <li class="tag-weight-2">
          <a href="/blog/jjameson/tags/Windows%207/default.aspx" title="Windows 7 (6)">
          Windows 7</a></li>
          <li class="tag-weight-2">
          <a href="/blog/jjameson/tags/Windows%20Server/default.aspx" title="Windows Server (28)">
          Windows Server</a></li>
          <li class="tag-weight-2">
          <a href="/blog/jjameson/tags/Windows%20Vista/default.aspx" title="Windows Vista (7)">
          Windows Vista</a></li>
          <li class="tag-weight-3">
          <a href="/blog/jjameson/tags/WSS%20v3/default.aspx" title="WSS v3 (63)">
          WSS v3</a></li>
          <li class="tag-weight-2">
          <a href="/blog/jjameson/tags/WSUS/default.aspx" title="WSUS (8)">
          WSUS</a></li>
          <li><a href="/blog/jjameson/tags/default.aspx">More...</a></li>
        </ul>
      </div>
      <div class="posts-categories">
        <h2>Categories</h2>
        <ul>
          <li><a href="/blog/jjameson/category/1.aspx">Development</a></li>
          <li><a href="/blog/jjameson/category/2.aspx">Infrastructure</a></li>
          <li><a href="/blog/jjameson/category/3.aspx">My System</a></li>
          <li><a href="/blog/jjameson/category/4.aspx">Personal</a></li>
          <li><a href="/blog/jjameson/category/5.aspx">SharePoint</a></li>
        </ul>
      </div>
      <div class="posts-archive">
        <h2>Archives</h2>
        <ul>
          <li>2011
          <ul>
            <li><a href="/blog/jjameson/archive/2011/09.aspx">September
            (2)</a></li>
            <li><a href="/blog/jjameson/archive/2011/08.aspx">August
            (1)</a></li>
            <li><a href="/blog/jjameson/archive/2011/05.aspx">May (3)</a></li>
            <li><a href="/blog/jjameson/archive/2011/04.aspx">April
            (10)</a></li>
            <li><a href="/blog/jjameson/archive/2011/03.aspx">March
            (22)</a></li>
            <li><a href="/blog/jjameson/archive/2011/02.aspx">February
            (6)</a></li>
            <li><a href="/blog/jjameson/archive/2011/01.aspx">January
            (2)</a></li>
          </ul>
          </li>
          <li>2010<ul>
            <li><a href="/blog/jjameson/archive/2010/12.aspx">December
            (8)</a></li>
            <li><a href="/blog/jjameson/archive/2010/11.aspx">November
            (4)</a></li>
            <li><a href="/blog/jjameson/archive/2010/10.aspx">October
            (2)</a></li>
            <li><a href="/blog/jjameson/archive/2010/09.aspx">September
            (2)</a></li>
            <li><a href="/blog/jjameson/archive/2010/08.aspx">August
            (2)</a></li>
            <li><a href="/blog/jjameson/archive/2010/07.aspx">July (3)</a></li>
            <li><a href="/blog/jjameson/archive/2010/06.aspx">June (3)</a></li>
            <li><a href="/blog/jjameson/archive/2010/05.aspx">May (24)</a></li>
            <li><a href="/blog/jjameson/archive/2010/04.aspx">April
            (19)</a></li>
            <li><a href="/blog/jjameson/archive/2010/03.aspx">March
            (12)</a></li>
            <li><a href="/blog/jjameson/archive/2010/02.aspx">February
            (1)</a></li>
            <li><a href="/blog/jjameson/archive/2010/01.aspx">January
            (8)</a></li>
          </ul>
          </li>
          <li>2009<ul>
            <li><a href="/blog/jjameson/archive/2009/12.aspx">December
            (6)</a></li>
            <li><a href="/blog/jjameson/archive/2009/11.aspx">November
            (15)</a></li>
            <li><a href="/blog/jjameson/archive/2009/10.aspx">October
            (28)</a></li>
            <li><a href="/blog/jjameson/archive/2009/09.aspx">September
            (16)</a></li>
            <li><a href="/blog/jjameson/archive/2009/08.aspx">August
            (3)</a></li>
            <li><a href="/blog/jjameson/archive/2009/06.aspx">June (12)</a></li>
            <li><a href="/blog/jjameson/archive/2009/05.aspx">May (3)</a></li>
            <li><a href="/blog/jjameson/archive/2009/04.aspx">April
            (6)</a></li>
            <li><a href="/blog/jjameson/archive/2009/03.aspx">March
            (22)</a></li>
            <li><a href="/blog/jjameson/archive/2009/02.aspx">February
            (2)</a></li>
            <li><a href="/blog/jjameson/archive/2009/01.aspx">January
            (2)</a></li>
          </ul>
          </li>
          <li>2008<ul>
            <li><a href="/blog/jjameson/archive/2008/11.aspx">November
            (1)</a></li>
            <li><a href="/blog/jjameson/archive/2008/10.aspx">October
            (2)</a></li>
            <li><a href="/blog/jjameson/archive/2008/09.aspx">September
            (1)</a></li>
            <li><a href="/blog/jjameson/archive/2008/08.aspx">August
            (1)</a></li>
            <li><a href="/blog/jjameson/archive/2008/07.aspx">July (1)</a></li>
            <li><a href="/blog/jjameson/archive/2008/06.aspx">June (2)</a></li>
            <li><a href="/blog/jjameson/archive/2008/05.aspx">May (4)</a></li>
            <li><a href="/blog/jjameson/archive/2008/04.aspx">April
            (8)</a></li>
            <li><a href="/blog/jjameson/archive/2008/02.aspx">February
            (3)</a></li>
            <li><a href="/blog/jjameson/archive/2008/01.aspx">January
            (1)</a></li>
          </ul>
          </li>
          <li>2007<ul>
            <li><a href="/blog/jjameson/archive/2007/11.aspx">November
            (2)</a></li>
            <li><a href="/blog/jjameson/archive/2007/10.aspx">October
            (4)</a></li>
            <li><a href="/blog/jjameson/archive/2007/08.aspx">August
            (1)</a></li>
            <li><a href="/blog/jjameson/archive/2007/06.aspx">June (6)</a></li>
            <li><a href="/blog/jjameson/archive/2007/05.aspx">May (6)</a></li>
            <li><a href="/blog/jjameson/archive/2007/04.aspx">April
            (3)</a></li>
            <li><a href="/blog/jjameson/archive/2007/03.aspx">March
            (8)</a></li>
          </ul>
          </li>
        </ul>
      </div>
    </div>
  </div>
</div>
<script type="text/javascript">
  $(".posts-archive>ul").collapseList();
</script>
</asp:Content>
```

> **Note**
>
> Observe how the prototype uses nested master pages in order to render the
> common masthead, global navigation, and footer content from Default.master in
> addition to the content specified in BlogPost.master.

### Sample blog post - new-blog-location.aspx

Here is a sample blog page from the Caelum prototype (referencing
BlogPost.master and specifying the content of the post in the **PostContent**
placeholder):

```
<%@ Page Title="" Language="C#" MasterPageFile="../../../../BlogPost.master" %>

<asp:Content ContentPlaceHolderID="PostContent" Runat="Server">
<div class="hentry">
  <h2 class="entry-title">New Blog Location - https://www.technologytoolbox.com/blog/jjameson
  </h2>
  <ul class="post-info">
    <li class="published"><span class="label">Published </span>
    <span class="value">September 2, 2011</span><span class="label"> at
    </span><span class="value">1:59 PM</span></li>
    <li class="vcard author">by <span class="fn">Jeremy Jameson</span></li>
    <li class="comments none"><a href="#postComments"><span class="label">Comments:
    </span><span class="value count">0</span></a></li>
    <li class="categories">
    <div class="post-categories">
      Categories:
      <ul>
        <li>
        <a href="/blog/jjameson/category/1.aspx" rel="tag" title="View all posts in this category">
        Personal</a></li>
      </ul>
    </div>
    </li>
  </ul>
  <div class="entry-content">
    <blockquote class="note original-post">
      <div class="noteTitle">
        <strong>Note</strong></div>
      <div>
        This post originally appeared on my MSDN blog:<br /><br />
        <div class="reference">
          <div class="referenceLink">
            <a href="http://blogs.msdn.com/b/jjameson/archive/2011/09/02/new-blog-location-http-www-technologytoolbox-com-blog-jjameson.aspx">
            http://blogs.msdn.com/b/jjameson/archive/2011/09/02/new-blog-location-http-www-technologytoolbox-com-blog-jjameson.aspx</a></div>
        </div>
        <p>Since
        <a href="/blog/jjameson/archive/2011/09/02/last-day-with-microsoft.aspx">
        I no longer work for Microsoft</a>, I have copied it here in case
        that blog ever goes away.</p>
      </div>
    </blockquote>
    <p>As I mentioned in my
    <a href="/blog/jjameson/archive/2011/08/22/leaving-microsoft.aspx">previous
    post</a>, today is my last day with Microsoft.</p>
    <p>I still have a lot of work to do in setting up my new website, but at
    this point, I've completed enough to continue blogging the "Random Musings
    of Jeremy Jameson":</p>
    <blockquote>
      <a href="https://www.technologytoolbox.com/blog/jjameson">https://www.technologytoolbox.com/blog/jjameson</a></blockquote>
    <p>If you subscribed to my MSDN blog in the past, then I encourage you to
    update your RSS feed by clicking the <strong>Subscribe</strong> link on
    my new blog.</p>
    <p>The blog will be in constant flux over the next week or so as I work
    on branding, layout, etc. So, as they say, please "pardon my dust" ;-)</p>
    <div class="tags">
      <h3>Tags</h3>
      <ul>
        <li><a href="/blog/jjameson/tags/Personal" rel="tag">Personal</a></li>
      </ul>
    </div>
  </div>
</div>
</asp:Content>
```

Figure 3 shows roughly what the corresponding page looks like when rendered in a
browser. [This screenshot is actually from the live site, but the prototype page
looks very similar and thus I didn't bother creating a separate screenshot for
it.]

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Technology-Toolbox-Blog-Post-398x600.png"
alt="Sample blog post" class="screenshot" height="600" width="398"
title="Figure 3: Sample blog post" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Technology-Toolbox-Blog-Post-1058x1595.png)

### Advantages of using a static HTML prototype

The primary benefit of creating a static HTML prototype is the ability to
rapidly throw together some sample HTML content and subsequently create the
corresponding CSS rules to style the pages. While you can certainly achieve the
same result using Visual Studio and your actual SharePoint or ASP.NET Web
project, it is typically much faster to do this using static HTML.

Similarly, when the time comes to develop some JavaScript for the site, it is
often easier to work against a static HTML file rather than an actual page
rendered in the context of the Web application. You can see some examples of
this in the code shown above (e.g. the jQuery used to collapse the lists in the
**Archives** section of the blog pages).

### Disadvantages of using a static HTML prototype

Using a prototype doesn't come without penalty. The biggest drawback is that you
have to be diligent about keeping the protoype in sync with the corresponding
ASP.NET or SharePoint solution. If you decide to change the structure of the
HTML emitted by your ASP.NET code or SharePoint Web Parts, you need to update
the prototype accordingly. However, this is all the more reason to really think
about the HTML your are producing and ensure that is is semantic and
well-structured from the start.

Likewise, whenever you tweak your CSS, you typically end up making the change in
two different files (e.g. $/Caelum/Dev/CaelumPrototype/Themes/Theme-1.0/Main.css
and $/Caelum/Main/Source/Website/Themes/Theme-1.0/Main.css).

The reality is that once the HTML for a particular feature is "baked", I rarely
have the need to change it. As for keeping the CSS files in sync, this actually
ends up being very easy -- since I structure the prototype and ASP.NET Web
application the same, the CSS files end up being identical. Thus I can simply
copy/paste the entire content of a CSS file from one location to another
immediately before I check-in a change.
