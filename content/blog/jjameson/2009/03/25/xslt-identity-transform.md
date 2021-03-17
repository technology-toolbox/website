---
title: "XSLT \"Identity Transform\""
date: 2009-03-25T07:58:00-06:00
excerpt: "Last week, I was explaining to a teammate that it is often helpful to use the XSLT \"Identity Transform\" in order to view the raw XML -- in other words, without any \"real\" transformation by the XSL stylesheet. 
 For example, suppose you need to customize..."
aliases: ["/blog/jjameson/archive/2009/03/24/xslt-identity-transform.aspx", "/blog/jjameson/archive/2009/03/25/xslt-identity-transform.aspx"]
draft: true
categories: ["SharePoint", "Development", "My System"]
tags: ["MOSS 2007", "Core Development", "Toolbox"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/25/xslt-identity-transform.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/25/xslt-identity-transform.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Last week, I was explaining to a teammate that it is often helpful to use the
XSLT "Identity Transform" in order to view the raw XML -- in other words,
without any "real" transformation by the XSL stylesheet.

For example, suppose you need to customize how search results are displayed in
Microsoft Office SharePoint Server (MOSS) 2007. With the Search Core Results Web
Part -- or really any Data View Web Part or the newer Data Form Web Part -- you
can temporarily replace the XSL stylesheet with the "Identity Transform" in
order to view the raw XML to be transformed.

This certainly isn't a new concept -- MSDN has shown how to do this since before
MOSS 2007 was even released. Perhaps it was just the fact that I referred to
this as the "Identity Transform" that was unknown to my colleague. Having an
engineering background -- with more coursework in matrices, determinants, and
systems of linear equations than I can remember -- it seems only natural to
refer to this as the "Identity Transform."

Regardless of what you call it, here is what I periodically use to view the raw
XML:

```
<xsl:stylesheet version="1.0"
  xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
  <xsl:output method="xml" version="1.0" encoding="UTF-8" indent="yes"/>
  <xsl:template match="/">
    <xmp>
      <xsl:copy-of select="*"/>
    </xmp>
  </xsl:template>
</xsl:stylesheet>
```

The deprecated `<xmp>` element is simply used to render the raw XML as text, not
as HTML-formatted elements (I don't recommend using this element in any
"permanent" fashion).

Save the stylesheet above in your
[Toolbox](/blog/jjameson/2007/03/22/backedup-and-notbackedup) as
**IdentityTransform-Formatted.xslt** and you'll never have to recall it from
memory or search the Web for it again.
