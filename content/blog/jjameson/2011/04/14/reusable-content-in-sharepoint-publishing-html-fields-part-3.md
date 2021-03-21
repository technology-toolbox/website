---
title: Reusable Content in SharePoint Publishing HTML Fields, Part 3
date: 2011-04-14T05:56:00-06:00
excerpt:
  In part 2 of this series , I explained how to programmatically add a new
  Reusable Content list item and subsequently add it to a Publishing HTML field
  on a page. I also provided a complete sample for SharePoint 2010 that
  demonstrates how this can be accomplished...
aliases:
  [
    "/blog/jjameson/archive/2011/04/13/reusable-content-in-sharepoint-publishing-html-fields-part-3.aspx",
    "/blog/jjameson/archive/2011/04/14/reusable-content-in-sharepoint-publishing-html-fields-part-3.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "SharePoint 2010"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2011/04/14/reusable-content-in-sharepoint-publishing-html-fields-part-3.aspx"
---

In
[part 2 of this series](/blog/jjameson/2011/04/13/reusable-content-in-sharepoint-publishing-html-fields-part-2),
I explained how to programmatically add a new **Reusable Content** list item and
subsequently add it to a Publishing HTML field on a page. I also provided a
complete sample for SharePoint 2010 that demonstrates how this can be
accomplished with minimal effort (thanks to some nitfy helper classes).

However, depending on your specific business requirements, *creating* a
SharePoint page that leverages reusable content may only be half the battle.
What if you need to get that content out of SharePoint?

For example, in
[part 1 of this series](/blog/jjameson/2011/04/08/reusable-content-in-sharepoint-publishing-html-fields-part-1),
I mentioned how I previously built a custom "document publishing" system based
on the WCM features in SharePoint. In the first sprint of that effort, one of
the core scenarios was exporting a SharePoint site (typically containing about a
hundred pages of content) to a PDF file. We call this the "Export to PDF"
feature. In the current sprint, I've been working on an enhancement to export
the documents to an external system (so clients can view a copy of the content
stored in that other system). We call this the "Export to {external system}"
feature. [I've substituted the name of the actual system with a placeholder to
protect the innocent ;-) ]

The "Export to PDF" feature was originally built with very little code. [You may
recall that I previously stated this solution had to be delivered to Production
in four weeks.]

Essentially, I wrote a little bit of code to "aggregate" all of the pages in a
specific SharePoint site into one large HTML document. The solution then uses
[Prince](http://www.princexml.com/) (in combination with some custom cascading
style sheets I created) to convert the HTML file into a commercial-quality PDF
file.

The "Export to PDF" process typically takes between 10-15 seconds (which seems
very reasonable given that some of the resulting PDF documents can exceed 100
printed pages). Consequently, I implemented this using a "long running page" in
SharePoint (you know, one of those application pages with the "spinning wheel"
that says something like "Please wait while...").

The most important aspect of the "Export to PDF" feature -- with regards to
reusable content -- is that it runs within the context of a SharePoint HTTP
request. In other words, when **SPContext.Current** is not null. During that
original sprint, I discovered that it is actually quite trivial to "expand" the
reusable content placeholders in Publishing HTML fields *when*
***SPContext.Current*** *is not null*. On the other hand, when
**SPContext.Current** *is* null, it takes a fair amount of custom code to
retrieve the same content. More on that in a moment.

Let's start with the simplest scenario first...

### "Expanding" reusable content placeholders when SPContext.Current is not null

When you need to get the HTML content from a SharePoint page and
SPContext.Current is not null (i.e. your code is running in the context of a
SharePoint HTTP request), then you can simply use the
[HtmlField.GetFieldValueAsHtml](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.publishing.fields.htmlfield.getfieldvalueashtml.aspx)
method, as shown below:

```C#
    HtmlField pageContentField =
        (HtmlField) page.Fields[FieldId.PublishingPageContent];

    string pageContent = pageContentField.GetFieldValueAsHtml(
        page.ListItem[FieldId.PublishingPageContent]);
```

If you look at this method with Reflector, you'll see that most of the work is
actually done by the
**[HtmlEditorInternal](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.publishing.internal.webcontrols.htmleditorinternal%28v=office.12%29.aspx)**
class.

If SPContext.Current is null, then a NullReferenceException is thrown in the
**[HtmlEditorInternal.ConvertStorageFormatToViewFormat](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.publishing.internal.webcontrols.htmleditorinternal_members%28v=office.12%29.aspx)**
method (due to an attempt to reference SPContext.Current.Site).

When developing the original "Export to PDF" solution, I implemented a hack to
insert a warning message in the underlying HTML when SPContext.Current is null.
[Note that I tend to write as much code as a I can outside the context of a Web
request -- for example, by writing unit tests that invoke the core services
underlying the presentation layer.]

The original code looked like this:

```C#
                if (pageContent.Contains("__publishingReusableFragment") == true)
                {
                    // HACK: We need to "expand" the reusable content in order
                    // to obtain the complete HTML page content. Unfortunately,
                    // SharePoint does not currently provide an easy way to do
                    // this outside the context of a Web request. Therefore
                    // just emit a warning message in the content when
                    // SPContext.Current is null (since attempting to replace
                    // the reusable content fragments ourselves would require
                    // a substantial amount of custom code).
                    if (SPContext.Current != null)
                    {
                        HtmlField pageContentField =
                            (HtmlField)page.Fields[FieldId.PublishingPageContent];

                        pageContent = pageContentField.GetFieldValueAsHtml(
                            pageContent);
                    }
                    else
                    {
                        writer.WriteLine(
@"<p><strong>Warning:</strong> The page content contains reusable content, which
    is not included when SPContext.Current is null. To obtain the full content
    -- including all reusable content -- submit the request within the context
    of a SharePoint site.</p>");
                    }
                }
```

This worked fine for the original release because we only needed to support the
"Export to PDF" feature in the "long running page" that I mentioned before. When
running my unit tests, the resulting PDF contained the warning message inserted
by the code above, but that was just fine with me.

However, as is often the case, the business requirements changed over time and
we needed to evolve the solution accordingly.

### "Expanding" reusable content placeholders when SPContext.Current is null

As I mentioned before, in addition to the original "Export to PDF" feature, we
now need the ability to export the documents to an external system. The problem
is that, unlike the process of exporting to PDF, exporting to an external system
isn't going to run in a matter of a few seconds. Rather, I estimated that it
could take several minutes to export a ~100 page document from SharePoint to the
external system. [Keep in mind that this is going over the Internet and we also
need to export inline images and referenced files from the SharePoint site to
the external system (in addition to the actual HTML content).]

Due to the expected latency of the export process, I decided to implement it
using a custom SharePoint workflow. This would allow it to run asynchronously
instead of within the context of a "long running page" (which would be
problematic if the user closed the browser while the export was still running).

The problem with running the export in a workflow, however, is that
SPContext.Current is null -- and unlike, my simple hack in the intial sprint, a
warning message simply wasn't going to cut it in this case ;-)

Consequently, I implemented a custom method in the **SharePointHtmlFieldHelper**
class for getting the "expanded" HTML content from a Publishing HTML field:

```C#
        public static string GetFieldValueAsHtml(
            SPWeb web,
            object value)
        {
            if (web == null)
            {
                throw new ArgumentNullException("web");
            }

            string str = value as string;

            if (string.IsNullOrEmpty(str) == true)
            {
                return string.Empty;
            }

            bool canCacheResults = true;

            return ConvertStorageFormatToViewFormat(
                web,
                str,
                out canCacheResults);
        }
```

If the method signatures look a little goofy, it's because I tried to keep them
as similar as possible to the out-of-the-box SharePoint classes. [My hope -- no
matter how small the probability -- is that the SharePoint code will be
eventually be improved to handle the scenario where SPContext.Current is null.
Let's just say that I'm not expecting that to happen, but that doesn't mean I
can't hope for more robust code from the SharePoint team.]

The reason why I require an SPWeb object is so I can subsequently read the
corresponding list items from the **Reusable Content** list. [Technically
speaking, I really only need an SPSite object, but I chose an SPWeb in order to
provide additional detail in cases where "malformed" content is detected (e.g.
the number of placeholders specified in the HTML does not match the number of
items in the reusable content "header").]

If you look at the code (see attachment on previous post), then you'll see that
I'm essentially just following the original implementation in
**HtmlEditorInternal** except that I don't rely on SPContext.Current and instead
use the specified SPWeb parameter to access the **Reusable Content** list.

{{< div-block "note important" >}}

> **Important**
>
> Also note that, unlike the original **HtmlEditorInternal** implementation, I
> don't leverage any caching when fetching **Reusable Content** list items.
> Consequently, you should be wary of using this implementation in very high
> volume scenarios. For the purposes of my current project, performance has
> proved to be more than adequate.

{{< /div-block >}}

I should point out, however, that I still prefer to use the out-of-the-box
SharePoint code whenever possible, which is why I provide a wrapper method in
the **SharePointPublishingHelper** class that determines whether or not it is
"safe" to use the **HtmlField.GetFieldValueAsHtml** method:

```C#
        /// <summary>
        /// Returns the value of the "Page Content" field for the specified page
        /// in HTML format. If the field contains any "reusable content"
        /// placeholders, they are expanded in order to return the HTML just as
        /// it would appear directly on the page.
        /// </summary>
        /// <param name="page">The page to get the content for.</param>
        /// <returns>The HTML content for the "Page Content" field.</returns>
        public static string GetPageContentAsHtml(
            PublishingPage page)
        {
            if (page == null)
            {
                throw new ArgumentNullException("page");
            }

            string pageContent = null;

            if (SPContext.Current != null)
            {
                // In this case, the out-of-the-box SharePoint functionality
                // works, so use it (rather than our own hack)
                HtmlField pageContentField =
                    (HtmlField) page.Fields[FieldId.PublishingPageContent];

                pageContent = pageContentField.GetFieldValueAsHtml(
                    page.ListItem[FieldId.PublishingPageContent]);

            }
            else
            {
                // HACK: In this case, the out-of-the-box SharePoint
                // functionality throws a NullReferenceException, so get the
                // HTML content using custom code
                pageContent = SharePointHtmlFieldHelper.GetFieldValueAsHtml(
                     page.PublishingWeb.Web,
                     page.ListItem[FieldId.PublishingPageContent]);
            }

            return pageContent;
        }
```

If you download the sample solution from my previous post, you can run all the
various unit tests to verify things are working as expected, including the
following unit test, which demonstrates why you can't use the
**HtmlField.GetFieldValueAsHtml** method when SPContext.Current is null:

```C#
        /// <summary>
        /// This test simply demonstrates why the out-of-the-box
        /// <see cref="Microsoft.SharePoint.Publishing.Fields.HtmlField.GetFieldValueAsHtml"/>
        /// method cannot be used to "expand" reusable content when
        /// SPContext.Current is null.
        /// </summary>
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Naming",
            "CA1707:IdentifiersShouldNotContainUnderscores")]
        [TestMethod()]
        [ExpectedException(typeof(NullReferenceException))]
        public void HtmlField_GetFieldValueAsHtml_FailsWhenSPContextCurrentIsNull()
        {
            string fabrikamDemoUrl = GetFabrikamDemoUrl();

            const string webUrl = "/";
            const string pageUrl = "Pages/ReusableContentSample.aspx";

            // The following HTML should not appear in the "storage" format but
            // should appear in the "view" format (in other words, when the
            // reusable content is "expanded")
            const string reusableContentFragment =
                "<h2>Statement of Non-Discrimination and Affirmative Action</h2>";

            using (SPSite site = new SPSite(fabrikamDemoUrl))
            {
                using (SPWeb web = site.OpenWeb(webUrl))
                {
                    PublishingPage page =
                        SharePointPublishingHelper.FindPublishingPage(
                            web,
                            pageUrl);

                    string pageContent = (string)page.ListItem[
                        FieldId.PublishingPageContent];

                    Assert.IsTrue(pageContent.Contains(
                        "__publishingReusableFragment"));

                    Assert.IsFalse(pageContent.Contains(
                        reusableContentFragment));

                    // Now demonstrate that HtmlField.GetFieldValueAsHtml
                    // throws NullReferenceException when SPContext.Current is
                    // null...
                    Assert.IsNull(SPContext.Current);

                    HtmlField pageContentField =
                        (HtmlField)page.Fields[FieldId.PublishingPageContent];

                    pageContentField.GetFieldValueAsHtml(
                        pageContent);
                }
            }
        }
```

If this unit test ever starts failing, I'll know there's a good chance that I
can scrap my custom code and go with the out-of-the-box SharePoint code instead
;-)

{{< div-block "note" >}}

> **Note**
>
> You must be running Visual Studio 2010 Service Pack 1 in order to run my
> SharePoint 2010 unit tests (in order to avoid bugs in 64-bit environments like
> the one I described in a
> [post from a couple of years ago](/blog/jjameson/2009/10/08/web-application-at-could-not-be-found-error-on-moss-2007-x64)).

{{< /div-block >}}
