---
title: Reusable Content in SharePoint Publishing HTML Fields, Part 1
date: 2011-04-08T06:59:00-06:00
excerpt:
  "In one of the sprints last year for my current project, I built a custom
  \"document publishing\" system based on the Web Content Management (WCM)
  features in Microsoft Office SharePoint Server (MOSS) 2007. My client was
  looking to replace a legacy system..."
aliases:
  [
    "/blog/jjameson/archive/2011/04/07/reusable-content-in-sharepoint-publishing-html-fields-part-1.aspx",
    "/blog/jjameson/archive/2011/04/08/reusable-content-in-sharepoint-publishing-html-fields-part-1.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "SharePoint 2010"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/04/08/reusable-content-in-sharepoint-publishing-html-fields-part-1.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/04/08/reusable-content-in-sharepoint-publishing-html-fields-part-1.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

In one of the sprints last year for my current project, I built a custom
"document publishing" system based on the Web Content Management (WCM) features
in Microsoft Office SharePoint Server (MOSS) 2007. My client was looking to
replace a legacy system used to create Microsoft Word documents that are
essentially "standard operating procedures" for their employees (or "SOPs" as we
called them many years ago when I worked at Gambro Healthcare).

While using SharePoint WCM features to create *documents* might seem a little
odd at first, it is important to understand that one of the key requirements for
the solution (besides delivering it to Production in four weeks) is that some
sections of the documents must be centrally managed (in other words, not
editable by the people creating the documents, but rather by a different team
entirely).

For example, section 13.0 of each and every document must contain the company's
standard "Sexual Harassment Statement" which essentially consists of several
paragraphs of legal text followed by some customizable text to specify which
managers are responsible for handling complaints regarding any form of
harassment.

When I heard about this requirement, I immediately thought of the Reusable
Content feature in SharePoint. The idea is that the Legal department specifies
the bulk of the content for section 13.0 (via a list item in the centralized
**Reusable Content** list). The document authors are only responsible for
specifying information about the managers designated to handle complaints (in
other words, the content that varies with each document).

Note that the **Reusable Content** list item has the **Automatic Update** field
set to **Yes**, as shown below. This is the key to "centrally managing" the
content. [Also note that in this particular solution, the document authors have
read-only access to the **Reusable Content** list.]

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/Reusable-Content-List-463x600.png"
alt="Reusable Content list" class="screenshot" height="600" width="463"
title="Figure 1: Reusable Content list" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/Reusable-Content-List-991x1285.png)

The following screenshot shows a sample page from one of the sites used to
create a document. As you can see, the HTML content from the list item above has
been inserted as expected into the page. [Note that I didn't bother to fill in
the section at the bottom of the page on my test site, so please ignore the
highlighted text.]

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/Reusable-Content-600x511.png"
alt="Reusable Content in \"view\" mode" class="screenshot" height="511"
width="600" title="Figure 2: Reusable Content in \"view\" mode" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/Reusable-Content-1006x857.png)

Note that when the document author is editing the page, SharePoint marks the
reusable content section as read-only, as shown below.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/Reusable-Content-edit-mode-600x423.png"
alt="Reusable Content in \"edit\" mode" class="screenshot" height="423"
width="600" title="Figure 3: Reusable Content in \"edit\" mode" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/Reusable-Content-edit-mode-1011x713.png)

One of the more interesting issues that I encountered when using the reusable
content feature in SharePoint is that when a user does not have access to the
corresponding list item in the **Reusable Content** list, the content is
silently removed from the page. This makes sense when you think about it, but
it's certainly a "gotcha" to be aware of.

For example, consider the following screenshot that shows the same page as
Figure 3. However, the following screenshot was taken *prior* to approving the
list item shown in Figure 1 above.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/Reusable-Content-missing-content-600x211.png"
alt="Page with missing content (due to \"Pending\" status in Reusable Content list)"
class="screenshot" height="211" width="600"
title="Figure 4: Page with missing content (due to \"Pending\" status in Reusable Content list)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/Reusable-Content-missing-content-1006x353.png)

> **Important**
>
> By default, the **Reusable Content** list is configured for content approval.
> Consequently when you add list items (either programmatically or through the
> site), the **Approval Status** defaults to **Pending**. In order for users
> with limited access to see the content as expected, a published version of the
> **Reusable Content** list item must be available.

Note that this post was primarily intended to introduce a scenario for using the
reusable content feature in SharePoint.

In
[my next post](/blog/jjameson/2011/04/13/reusable-content-in-sharepoint-publishing-html-fields-part-2),
I'll show you how to programmatically add **Reusable Content** list items (which
is very helpful when deploying to multiple environments, such as DEV, TEST, and
PROD).

In
[part 3 of this series](/blog/jjameson/2011/04/14/reusable-content-in-sharepoint-publishing-html-fields-part-3),
I'll show you how to "expand" the reusable content placeholders within, for
example, the Page Content field (essentially just two lines of code), as well as
some of the "gotchas" with the out-of-the-box solution (including a workaround).
