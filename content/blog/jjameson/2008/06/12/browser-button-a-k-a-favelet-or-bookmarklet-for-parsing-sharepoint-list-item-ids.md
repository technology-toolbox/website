---
title: Browser Button (a.k.a. Favelet or Bookmarklet) for Parsing SharePoint List Item IDs
date: 2008-06-12T08:06:00-06:00
excerpt:
  'This morning I received an email from a customer inquiring about making it
  easier for users to determine the unique identifier for each document in a
  library. Typically, users don''t really care about this "List Item ID" for a
  document, but there are scenarios...'
aliases:
  [
    "/blog/jjameson/archive/2008/06/11/browser-button-a-k-a-favelet-or-bookmarklet-for-parsing-sharepoint-list-item-ids.aspx",
    "/blog/jjameson/archive/2008/06/12/browser-button-a-k-a-favelet-or-bookmarklet-for-parsing-sharepoint-list-item-ids.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3", "WSS v2"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2008/06/12/browser-button-a-k-a-favelet-or-bookmarklet-for-parsing-sharepoint-list-item-ids.aspx"
---

This morning I received an email from a customer inquiring about making it
easier for users to determine the unique identifier for each document in a
library. Typically, users don't really care about this "List Item ID" for a
document, but there are scenarios where this is useful (or perhaps even
required).

My initial suggestion was to inform users to obtain the ID from the item
properties URL. For example, if you are viewing the properties of a document
([http://server/Library/Forms/DispForm.aspx?ID=1956](http://server/Library/Forms/DispForm.aspx?ID=1956))
then you can inspect the URL and determine that **1956** is the ID. Okay, I'll
admit, this gets a little cumbersome depending on the URL. For example, it
certainly isn't as easy to extract ID **2531** from:

```Text
http://server/sites/foobar/Lists/Work%20Items/EditForm.aspx?ID=2531&Source=http%3A%2F%2Fserver%2Fsites%2Ffoobar%2FLists%2FWork%2520Items%2FOpenItems%2Easpx
```

My alternate suggestion -- if the former was deemed unacceptable -- was to
display the ID on the item properties page by modifying the item "View"
template, similar to changes we had previously made for the "Edit" template in
order to render custom field controls. I wasn't wild about this idea, but it was
the only other option I could see at the time.

Sure enough, shortly after I clicked **Send** to respond to the email, a third
option occurred to me. How about using a tiny amount of JavaScript to do the
work of parsing the URL?

Now, if you have ever worked with me before, you probably know that I hate
JavaScript, and it takes some serious motivation to get me to start coding in
what I personally consider to be a rather arcane programming language. [No
offense to the hundreds of thousands of JavaScript developers out there. If it
makes you feel any better, I hate VBScript just as much. It must just be the
ex-C++ developer in me that simply can't get over the "loosey-goosey" nature of
scripting languages.]

The thought of using JavaScript occurred to me because I recently started using
the new [Social Bookmarks](http://social.msdn.microsoft.com/bookmarks) feature
on MSDN, and one of the features of this is a "browser button" (a.k.a. favelet
or bookmarklet) that makes it very easy to add the current page as a "social
bookmark."

With dreams of a creating a "killer utility for SharePoint", I set out to write
a little code to do the work of parsing the ID from various forms of the URL.
Here is what I came up with:

> [Parse List Item ID](javascript:s=location.href;pos1=s.indexOf%28'DispForm.aspx?ID=',%200%29;if%28pos1==-1%29{window.alert%28'Unable%20to%20determine%20List%20Item%20ID%20from%20URL.'%29;}else{pos1+='DispForm.aspx?ID='.length;pos2=s.indexOf%28'&',%20pos1%29;if%28pos2==-1%29{pos2=s.length;}listItemIntId=s.substr%28pos1,%20pos2-pos1%29;window.alert%28'List%20Item%20ID:%20'%20+%20listItemIntId%29;})

With all due credit to the MSDN team, here is some more detail that I snarfed
from their [FAQ page](http://social.msdn.microsoft.com/bookmarks/en-US/FAQ)
(with minor modifications, of course):

### What is a browser button?

A browser button (also called a bookmarklet) is an easy way to add bookmarks.
The button adds a link to your web browser containing JavaScript. (Don't worry!
This script is safe to include in your browser).

### How do I install the browser button?

For Internet Explorer users:

1. Right click the **Parse List Item ID** link and choose **Add to Favorites**.
1. A security alert dialog will warn you that the link may be unsafe. Click
   **Yes**.
1. In the **Save in** dropdown list, choose **Links**. Click **Add**.

For Firefox users:

1. Drag the **Parse List Item ID** link to your Links/Bookmarks toolbar.

### How do I uninstall the browser button?

For Internet Explorer users:

1. Right click the **Parse List Item ID** button on the Favorites toolbar and
   choose **Delete**.
1. A confirmation dialog will warn you that the button will be permanently
   deleted. Click **Yes**.

For Firefox users:

1. Right click the **Parse List Item ID** button on the Links/Bookmarks toolbar
   and choose **Delete**.
1. A confirmation dialog will warn you that the button will be permanently
   deleted. Click **Yes**.
