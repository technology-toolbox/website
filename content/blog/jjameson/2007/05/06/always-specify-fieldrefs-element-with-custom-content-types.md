---
title: Always Specify <FieldRefs> Element with Custom Content Types
date: 2007-05-06T06:57:00-06:00
excerpt:
  I came across a nasty bug earlier this week in Microsoft Office SharePoint
  Server (MOSS) 2007 -- especially nasty because troubleshooting it primarily
  involved trial and error since there were no error messages displayed on the
  page, in the SharePoint...
aliases:
  [
    "/blog/jjameson/archive/2007/05/05/always-specify-fieldrefs-element-with-custom-content-types.aspx",
    "/blog/jjameson/archive/2007/05/06/always-specify-fieldrefs-element-with-custom-content-types.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/05/06/always-specify-fieldrefs-element-with-custom-content-types.aspx"
---

I came across a nasty bug earlier this week in Microsoft Office SharePoint
Server (MOSS) 2007 -- especially nasty because troubleshooting it primarily
involved trial and error since there were no error messages displayed on the
page, in the SharePoint log, or in the Windows event logs. I even attached the
debugger and set it to break on all exceptions, but had no luck there either.

The problem was that the document properties form no longer rendered for our
custom content types, or rather that it would only *partially* render. The
toolbar was there, as was the footer displaying the content type, version, date
created, and date modified -- just no fields whatsoever.

It turns out that when I "rebuilt" our ContentTypes.xml file to include all of
our final content types, I omitted the empty `<FieldRefs>` elements that had
previously been there. For example:

```
  <ContentType ID="0x0101009F5C14F1CF5847c7BBBADA9A8637DEAB0106"
    Name="Brochure"
    Description=""
    Group="Fabrikam Content Types"
    Version="0">
  </ContentType>
```

instead of

```
  <ContentType ID="0x0101009F5C14F1CF5847c7BBBADA9A8637DEAB0106"
    Name="Brochure"
    Description=""
    Group="Fabrikam Content Types"
    Version="0">
    <FieldRefs>
    </FieldRefs>
  </ContentType>
```

If you do not specify the `<FieldRefs>` element, then your documents will not
inherit any properties from the parent content type (and the document properties
form will consequently appear to be blank). Ouch.

I found nothing in the SDK that indicates `<FieldRefs>` is a required element,
and according to wss.xsd, this is optional (i.e. `minOccurs="0"`).

You might be wondering why we would have a content type that does not specify
any additional fields. This is the result of a decision we made early on for our
solution. I am currently helping a customer migrate from a legacy document
management system that has over 100 "publication types." After a little debating
over whether to create distinct content types for each publication type or to
minimize the number of content types and instead have a "Publication Type"
field, we chose the former because we perceive it as being more flexible and
extensible in the future. Just because "Brochure" does not specify any
additional fields right now doesn't mean that it will never have any custom
fields.

Hopefully this post will save someone the hours I spent investigating this
issue.
