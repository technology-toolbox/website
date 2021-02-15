---
title: "Finding the Content Type ID in MOSS 2007"
date: 2009-09-29T23:45:00-07:00
excerpt: "Yesterday I received the following question from someone regarding Microsoft Office SharePoint Server (MOSS) 2007 content type IDs: 
 I need to add another page type [that] inherits from the article page. How do you find the GUID of the article page..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/09/30/finding-the-content-type-id-in-moss-2007.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/09/30/finding-the-content-type-id-in-moss-2007.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

Yesterday I received the following question from someone regarding Microsoft Office SharePoint Server (MOSS) 2007 content type IDs:

> I need to add another page type [that] inherits from the article page. How do you find the GUID of the article page to add it to the solution?

Here's the easiest way I know to find the content type ID:

1. On the home page of the site, click **Site Actions**, point to **Site Setttings**, and then click **Modify All Site Settings**.
2. On the **Site Settings** page, under the **Galleries** section, click **Site content types**.
3. On the **Site Content Type Gallery** page, click the content type that you need to find the unique identifier for, such as **Article page**.
4. On the **Site Content Type: {content type name}** page, copy the value of the **ctype** query string parameter. For example, the content type ID for **Article page** is 0x010100C568DB52D9D0A14D9B2FDCC96666E9F2007948130EC3DB064584E219954237AF3900242457EFB8B24247815D688C526CD44D.

Note that you can also retrieve the content type ID directly from the feature XML file (e.g. %ProgramFiles%\Common Files\microsoft shared\Web Server Extensions\12\TEMPLATE\FEATURES\PublishingResources\PublishingContentTypes.xml). However, I think the first approach is faster.

For more information on content type IDs in MOSS 2007, refer to the following MSDN article:

<cite>Content Type IDs</cite>
[http://msdn.microsoft.com/en-us/library/aa543822.aspx](http://msdn.microsoft.com/en-us/library/aa543822.aspx)
