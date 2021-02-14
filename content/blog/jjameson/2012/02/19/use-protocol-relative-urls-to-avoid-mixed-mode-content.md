---
title: "Use protocol-relative URLs to avoid mixed mode content"
date: 2012-02-19T23:39:17+08:00
excerpt: "Here's a great tip I picked up from Phil Haack a few weeks ago for avoiding those pesky warnings like \"Only secure content is displayed.\""
draft: true
categories: ["Development", "My System"]
tags: ["Simplify", "Web Development"]
---

A few weeks ago I added[some bug fixes for the Subtext blog engine](/blog/jjameson/archive/2012/01/31/building-technologytoolbox-com-part-19.aspx) to GitHub. One of the bugs was related to mixed mode content:

<cite>jQuery references need to specify "https://" (not "http://") when 	browsing Subtext pages using HTTPS</cite>
[http://github.com/Haacked/Subtext/issues/18](http://github.com/Haacked/Subtext/issues/18)


My initial fix for this issue was to change references like:



    <script type="text/javascript" src="http://ajax.googleapis.com/.../jquery.min.js"></script>



...to detect a secure connection (in which case, specify "https://" instead):



    <script type="text/javascript" src="<%= Request.IsSecureConnection ? "https" : "http" %>://ajax.googleapis.com/.../jquery.min.js"></script>



While this worked, Phil[pointed out](http://github.com/Haacked/Subtext/pull/7) there is a more elegant solution:


> Actually, we should simply use a
> 	[protocol 
> 	relative URL/network path reference](http://paulirish.com/2010/the-protocol-relative-url/).
> 
> For example:
> 
> 
> 		src="//ajax.googleapis.com/..."
> 
> That removes the need do use &lt;%= %&gt; blocks here and is cleaner.


I responded that I didn't even know that was possible (before reading Phil's comment, I would have assumed a URL beginning with "//" would be interpreted as a server-relative URL).

Consequently I updated my fix to incorporate Phil's recommendation:



    <script type="text/javascript" src="//ajax.googleapis.com/.../jquery.min.js"></script>



This just shows that there is always a valuable lesson to be learned each and every day -- especially in the world of software.

So, why do I bother to blog about this topic this morning? Well, I just discovered a similar bug on the [Search page](/Search.aspx) of TechnologyToolbox.com when accessing the site via HTTPS, due to the script reference for Google Site Search:



    <script src='http://www.google.com/jsapi' type='text/javascript'></script>



The bug fix for TechnologyToolbox.com will be included in the next release to PROD.

Here are some good resources for understanding more about protocol-relative URLs (as pointed out in the first resource below, these are technically called network-path references according to RFC 3986, but "protocol-relative URL" seems more intuitive):

- [The protocol-relative 	URL](http://paulirish.com/2010/the-protocol-relative-url/)
- [Using Protocol Relative URLs to Switch between HTTP and HTTPS](http://blog.httpwatch.com/2010/02/10/using-protocol-relative-urls-to-switch-between-http-and-https/)


Apparently there are some known issues in Internet Explorer 6 with protocol-relative URLs (no surprise there). However, I honestly couldn't care less about IE6 these days. Especially since I occasionally struggle with issues in IE9 that don't occur in Firefox and Chrome -- such as the one pointed out in[my previous post](/blog/jjameson/archive/2012/02/19/html-to-pdf-converters.aspx).

