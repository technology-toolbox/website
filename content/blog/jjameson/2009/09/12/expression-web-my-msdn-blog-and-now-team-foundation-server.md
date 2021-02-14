---
title: "Expression Web, My MSDN Blog, and (Now) Team Foundation Server"
date: 2009-09-12T03:17:00+08:00
excerpt: "In case you haven't picked it up from some of my previous posts, I became somewhat of a \"Web standards zealot\" back in 2006 while creating a \"community site\" for a local organization of mental health professionals and attorneys that help children and..."
draft: true
categories: ["My System", "Development"]
tags: ["My System", "Web Development", "TFS"]
---

> **Note**
> 
>             This post originally appeared on my MSDN blog:  
>   
> 
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/09/12/expression-web-my-msdn-blog-and-now-team-foundation-server.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/09/12/expression-web-my-msdn-blog-and-now-team-foundation-server.aspx)
> 
> 
> Since [I no longer work for Microsoft](/blog/jjameson/archive/2011/09/02/last-day-with-microsoft.aspx), I have copied it here in case that blog                 ever goes away.


In case you haven't picked it up from some of my previous posts, I became somewhat         of a "Web standards zealot" back in 2006 while creating a "community site" for a         [local organization of mental health professionals
            and attorneys](http://www.metrodenveridc.org/) that help children and their parents through divorce. After         reading books like [The Zen of CSS Design](http://amzn.com/0321303474),         [Web Standards Solutions](http://amzn.com/1430219203), and [Bulletproof Web Design](http://amzn.com/0321509021), I realized the fundamental concept of [Web standards](http://en.wikipedia.org/wiki/Web_standards) design closely aligns with my preference of keeping things         as simple as possible. Last year, I got a copy of [Transcending CSS](http://amzn.com/0321410971) and I highly recommend this book as well for anyone responsible         for creating Web sites.

Although it may strike people as somewhat odd, I've been using Microsoft Expression         Web for the last couple of years to manage the content for my MSDN blog. While the         underlying platform for MSDN blogs (i.e. Community Server) certainly provides rich         editing functionality, I personally prefer to have very tight control over the HTML         whenever I am creating content for the Web. There are also a few things that really         bother me about the editing functionality in Community Server (or at least the version         of Community Server that Microsoft is currently using).

Before finally choosing Expression Web, I looked at various other tools for authoring         blog posts, including Windows Live Writer and even Microsoft Word 2007. However,         I found that by using Expression Web and a simple "template" that I created, I could         quickly and easily create blog posts with just the kind of "minimal markup" that         I was looking for.

My template simply contains a variety of markup that I commonly use when creating         blog posts, such as:


    <blockquote class="directQuote errorMessage">
        [Direct quote, error message]</blockquote>


Whenever I need to create a new blog post, I simply copy my template (i.e. default.htm)         into the corresponding Archive\{year}\{month}\{day} folder, rename the file to something         like **sharepoint-2010-sneak-peek.aspx**, and then start replacing         the placeholder text with the corresponding content. Note that I have defined a         few custom CSS rules in Community Server to change the style of my blog content,         such as:


    .directQuote {
        font-style: italic;
    }
    .errorMessage {
        color: red;
    }


I define these styles in a CSS file (i.e. Themes\MSDN\MSDN.css) that is referenced         by each page, thus giving me a WYSIWYG (What You See Is What You Get) experience         when authoring blog content. This is one of the biggest things lacking from the         editing functionality provided by Community Server, meaning that it doesn't apply         your custom CSS rules when previewing a blog post before publishing.

Once I have finished writing a blog post in Expression Web, I simply copy the HTML         within the `<body>` element, browse to my MSDN blog, click the         **Write a Blog Post **link, click the **Edit HTML Source **         toolbar button in the Community Server Control Panel, and then paste my content         into the HTML source view.

I freely admit this approach takes a little longer than simply authoring posts directly         on the site, but I believe it is well worth it. Even more so, now that my blog is         stored in Team Foundation Server (TFS)...

Back in August while I was in Redmond for SharePoint 2010 Training, I took the opportunity         of being on the Microsoft campus to rebuild my laptop with Windows 7. Prior to that         rebuild, I had been running Windows Server 2008 on my laptop so that I could utilize         Hyper-V regardless of whether I was working at a customer site or from home (with         access to the ["Jameson Datacenter"](/blog/jjameson/archive/2009/09/14/the-jameson-datacenter.aspx) and all of my various VMs).

As part of this latest rebuild, I installed the latest version of the Microsoft         Expression suite. One of the new features I discovered in Expression Web 3 is the         ability to integrate with Team Foundation Server for source control. Note that this         integration isn't completely out-of-the-box (i.e. you have to download a [QFE](http://code.msdn.microsoft.com/KB967483) to enable it).

This morning, I finally found some time to move the "offline copy" of my MSDN content         from my Documents folder into Team Foundation Server. I simply needed to create         a new team project in TFS and then configure my workspace for source control. Once         this was done, the TFS integration features of Expression Web 3 simply just worked,         as shown below:

![Expression Web - my MSDN blog](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Expression%20Web%20-%20My%20MSDN%20Blog.png)
            Figure 1: Expression Web - my MSDN blog

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Expression%20Web%20-%20My%20MSDN%20Blog.png)


Notice how items that are checked into source control have a lock icon and new items         that are pending have a "plus" icon next to them. In other words, the Folder List         in Expression Web (which essentially shows your Web site "solution") now behaves         a lot like the Solution Explorer window in Visual Studio (at least from the perspective         of source control).

Even though the offline content of my MSDN blog content was previously backed up         on one of the servers in the Jameson Datacenter, now that it is stored in TFS, it         is much easier for me to author blog posts from multiple computers (by simply doing         a Get Latest from TFS). I also like the fact that my posts now have versioning --         so in the rare case where I actually go back and annotate a blog post, I can now         easily "diff" my changes whenever I want.

I personally love Expression Web, and now that it integrates with TFS, I love it         even more! [Although I'm still getting used to the new "dark theme" introduced in         v3 -- which was carried over from the previous version of Expression Blend.]

