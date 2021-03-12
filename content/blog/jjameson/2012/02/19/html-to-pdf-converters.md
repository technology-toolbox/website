---
title: "HTML-to-PDF converters"
date: 2012-02-19T03:25:09-07:00
lastmod: 2012-02-19T03:37:46-07:00
excerpt: "Looking for a solution to convert from HTML to PDF? Here is a list of the products I discovered during my research as well as the results of the head-to-head competition."
aliases: ["/blog/jjameson/archive/2012/02/18/html-to-pdf-converters.aspx", "/blog/jjameson/archive/2012/02/19/html-to-pdf-converters.aspx"]
draft: true
categories: ["Development", "My System", "SharePoint"]
tags: ["MOSS 2007", "SharePoint 2010", "Web Development"]
---

Here is one more quick post this morning so I can get rid of yet another
[item on my Windows desktop](/blog/jjameson/2012/02/18/stop-putting-shortcuts-on-my-windows-desktop)
that I should have deleted a long time ago -- this time a small Excel worksheet
that I created back in late 2010. [Wow, where does the time go?]

I mentioned in
[a previous post](/blog/jjameson/2011/04/14/reusable-content-in-sharepoint-publishing-html-fields-part-3)
how I created a custom "document publishing" system for a client based on the
Web Content Management features in SharePoint, which included an "Export to PDF"
feature.

While researching HTML-to-PDF converters, I created an Excel worksheet to
capture some notes about various products, pricing, etc. This formed the basis
for providing my recommendation to the client (Securitas).

Rather than attaching the Excel worksheet to this post, I'll simply copy the
content into a table.

> **Warning - Internet Explorer Users**
>
> While publishing this post, I discovered the following table looks really ugly
> in IE9 (since the width is not properly constrained to the content area). I
> spent no less than 20 minutes trying to force IE to make it look as nice as it
> does in Firefox and Chrome (e.g. by adding a **width** attribute to the table
> and even explicitly specifying column widths using a **colgroup** element.
> However, none of those hacks worked.
>
> While I would normally invest more effort trying to find a workaround for this
> bug in IE, I've got more important things to do this morning. If you are using
> IE and you find the following table difficult to read, may I suggest using a
> ~~better~~ different browser (e.g. Firefox)?

{{< table class="small" caption="HTML-to-PDF Converters" >}}

| Product | URL | Pricing | Comments |
| --- | --- | --- | --- |
| Adlib Software EXPRESS | [http://www.adlibsoftware.com](http://www.adlibsoftware.com/) | Priced by server, starting at $20-24k | Uses Amyuni libraries (according to Gabor Fari); extensive SharePoint integration (for document conversion); major focus on life sciences and other industries with strict regulatory requirements |
| Altsoft Xml2PDF | [http://www.alt-soft.com](http://www.alt-soft.com/) | Professional - $2,099 (includes 1 year maintenance); $550/yr maintenance | Built on .NET platform; company based in Belgium |
| Amyuni PDF Converter | [http://www.amyuni.com](http://www.amyuni.com/) | $459 per developer; $689 per application; $399/yr maintenance | Conversion appears to be through "print to PDF" (even for server applications -- e.g. "lock"/"unlock") |
| Antenna House Formatter v5 | [http://www.antennahouse.com](http://www.antennahouse.com/) | Server - $5,000 for first CPU, $4,000 each additional proc; $3,000 Developer license | Company Web site looks very elementary |
| Ibex PDF Creator | [http://www.xmlpdf.com](http://www.xmlpdf.com/) | $999 per developer (includes 1 year maintenance); $350/yr maintenance | "Team licensing" available for groups of ten or more developers; separate .NET and Java versions |
| Prince XML | [http://www.princexml.com](http://www.princexml.com/) | $3,800 per server; $950/yr maintenance | Current version is 7.1 (May 2010); original 1.0 version released in April 2003; support appears to be through email only |
| RenderX XEP | [http://www.renderx.com](http://www.renderx.com/) | $4,400.00 per CPU core | "written in Java" (http://www.renderx.com/tools/xep.html) |
| XSL:FO + Apache FOP | [http://xmlgraphics.apache.org/fop](http://xmlgraphics.apache.org/fop) | Free | FOP is Java-based |

{{< /table >}}

> **Note**
>
> The pricing shown above has likely changed in the time since I gathered the
> data, and there may very well be additional products available today that I
> did not come across during my research.

My intent for this post is not to provide an exhaustive list of the current
options for converting HTML to PDF. Rather I simply wanted to share the
information that I captured just in case it proves useful to others. [Something
I should have done long ago.]

Based on platform, suitability for a "server-side" solution, and pricing, we
decided to evaluate Altsoft Xml2PDF, Ibex PDF Creator, and Prince XML. Of those
three, I found Prince XML to be the most robust in terms of CSS support and
consequently recommended that product to Securitas.

One of the other products I evaluated (honestly, I can't remember which one) had
a strong tendency to crash when I tested it against the sample content and CSS
files that I developed. The other got some of the formatting right during the
conversion, but left a lot to be desired.

Prince, on the other hand, was very good at rendering the PDF nearly identical
to the on-screen version of the HTML content.

