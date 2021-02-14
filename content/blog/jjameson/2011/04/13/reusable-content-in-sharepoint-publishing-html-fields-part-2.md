---
title: "Reusable Content in SharePoint Publishing HTML Fields, Part 2"
date: 2011-04-13T12:06:00+08:00
excerpt: "In my previous post , I introduced a scenario for using the \"Reusable Content\" feature in Microsoft Office SharePoint Server (MOSS) 2007 and SharePoint Server 2010. In this post, I show you how to programmatically add Reusable Content list items (which..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "SharePoint 
		2010"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2011/04/13/reusable-content-in-sharepoint-publishing-html-fields-part-2.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/04/13/reusable-content-in-sharepoint-publishing-html-fields-part-2.aspx)
> 
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

In [my previous post](/blog/jjameson/2011/04/08/reusable-content-in-sharepoint-publishing-html-fields-part-1), I introduced a scenario for using the "Reusable Content" feature  in Microsoft Office SharePoint Server (MOSS) 2007 and SharePoint Server 2010. In  this post, I show you how to programmatically add **Reusable Content
**list items (which is very helpful when deploying to multiple environments,  such as DEV, TEST, and PROD) as well as how to insert reusable content into a Publishing  HTML field on a page (e.g. the **Page Content **field). I also provide  a complete code sample for SharePoint 2010 that demonstrates the key concepts discussed  in this series.

### Adding a new Reusable Content list item

Since reusable content in SharePoint is enabled by the Publishing features, I  chose to enhance the **SharePointPublishingHelper** class -- which  I introduced in a [previous post](/blog/jjameson/2009/10/09/introducing-the-sharepointpublishinghelper-class) -- to provide a new method for ensuring some piece of reusable  content has been configured.

In case you are not familiar with the "ensure" logic that I tend to implement  in various SharePoint helper classes, in essence the code performs any necessary  actions to achieve some condition (for example, ensure a specific page exists --  by creating a new page if one doesn't already exist). The key design principle in  the "ensure" logic is to avoid trampling any manual changes that may have been made  to SharePoint by content authors and administrators.

When it comes to adding a piece of reusable content, this "ensure" logic means  that we only want to configure the **Reusable Content **list item if  it doesn't already exist. In other words, we don't want to overwrite the content  if the list item already exists, because it may have been updated by a content author  or administrator since it was originally created (presumably upon activation of  some feature). [There are scenarios where you *do* want to overwrite the  existing site configuration, but those are beyond the scope of this post.]

For example, consider the following method of **SharePointPublishingHelper**:

```
public static SPListItem EnsureReusableContentItem(
            SPSite site,
            string title,
            bool automaticUpdate,
            string reusableHtml)
```

Imagine that you run the following code upon activation of a feature:

```
SPListItem reusableContent =
                SharePointPublishingHelper.EnsureReusableContentItem(
                    web.Site,
                    "Copyright",
                    true,
                    "Copyright&copy; 2009 Contoso Corporation - All Rights Reserved");
```

If the specified **Reusable Content **list item already exists (found  by matching on the **Title **field), then the existing list item is  returned. Otherwise, a new list item is added (with the specified field values)  and subsequently returned. Note that we don't want to overwrite the content in the  list item because someone may have updated the copyright content (e.g. "Copyright&copy;  2011 Fabrikam Technologies - All Rights Reserved").

Also note that the **Reusable Content **list is configured for approval  by default, and -- as I mentioned in [part 1 of this series](/blog/jjameson/2011/04/08/reusable-content-in-sharepoint-publishing-html-fields-part-1) -- "bad things" happen when a page includes a reference  to reusable content that has not been approved. Consequently, the **EnsureReusableContentItem
**method also takes care of approving the list item (if it does not have  at least one approved version):

```
public static SPListItem EnsureReusableContentItem(
            SPSite site,
            string title,
            bool automaticUpdate,
            string reusableHtml,
            string comments,
            string contentCategory)
        {
            if (site == null)
            {
                throw new ArgumentNullException("site");
            }

            if (title == null)
            {
                throw new ArgumentNullException("title");
            }

            title = title.Trim();
            if (string.IsNullOrEmpty(title) == true)
            {
                throw new ArgumentException(
                    "The title must be specified.",
                    "title");
            }

            if (reusableHtml == null)
            {
                throw new ArgumentNullException("reusableHtml");
            }

            reusableHtml = reusableHtml.Trim();
            if (string.IsNullOrEmpty(reusableHtml) == true)
            {
                throw new ArgumentException(
                    "The reusable HTML must be specified.",
                    "reusableHtml");
            }

            // Note: comments and contentCategory may be null
            
            SPLogger.Log(
                LogCategory.Configuration,
                TraceSeverity.Medium,
                "Configuring reusable content item ({0}) on site ({1})...",
                title,
                site.Url);

            if (string.IsNullOrEmpty(contentCategory) == true)
            {
                SPLogger.Log(
                    LogCategory.Configuration,
                    TraceSeverity.Medium,
                    "The content category was not specified, using default"
                        + " value (None)",
                    title);

                contentCategory = "None";
            }

            const string rootFolderUrl = "ReusableContent";

            SPList reusableContentList = SharePointListHelper.FindListByRootFolderUrl(
                site.RootWeb.Lists,
                rootFolderUrl);

            if (reusableContentList == null)
            {
                string message = string.Format(
                    CultureInfo.CurrentCulture,
                    "The list ({0}) could not be found on the site ({1}).",
                    rootFolderUrl,
                    site.Url);

                throw new InvalidOperationException(message);
            }

            EnsureReusableContentCategoryExists(reusableContentList, contentCategory);

            string camlQuery =
                "<Where><Eq><FieldRef Name='Title'/><Value Type='Text'>"
                    + title + "</Value></Eq></Where>";

            SPListItem listItem = SharePointListHelper.FindUniqueListItem(
                reusableContentList,
                camlQuery);

            if (listItem == null)
            {
                SPLogger.Log(
                    LogCategory.Configuration,
                    TraceSeverity.Medium,
                    "Adding reusable content item ({0}) to site ({1})...",
                    title,
                    site.Url);

                listItem = reusableContentList.Items.Add();
                listItem[SPBuiltInFieldId.Title] = title;
                listItem[FieldId.AutomaticUpdate] = automaticUpdate;
                listItem[FieldId.ReusableHtml] = reusableHtml;
                listItem[SPBuiltInFieldId.Comments] = comments;
                listItem[FieldId.ReusableTextType] = contentCategory;

                listItem.Update();

                SPLogger.LogEvent(
                    LogCategory.Configuration,
                    EventSeverity.Information,
                    "Successfully added reusable content item ({0}) to site"
                        + " ({1}).",
                    title,
                    site.Url);
            }

            if (listItem.HasPublishedVersion == false)
            {
                SPLogger.Log(
                    LogCategory.Configuration,
                    TraceSeverity.Medium,
                    "The reusable content item ({0}) does not have a published"
                        + " version. Approving list item ({1}/{2})...",
                    title,
                    listItem.Web.Url,
                    listItem.Url);

                listItem.ModerationInformation.Status =
                    SPModerationStatusType.Approved;

                listItem.Update();

                SPLogger.LogEvent(
                    LogCategory.Configuration,
                    EventSeverity.Information,
                    "Successfully approved reusable content item ({0})"
                        + " ({1}/{2}).",
                    title,
                    listItem.Web.Url,
                    listItem.Url);
            }
            else
            {
                SPLogger.Log(
                    LogCategory.Configuration,
                    TraceSeverity.Medium,
                    "The reusable content item ({0}) already has a"
                        + " published version and may have been customized, so"
                        + " no changes will be made to the list item"
                        + " ({1}/{2}).",
                    title,
                    listItem.Web.Url,
                    listItem.Url);
            }

            return listItem;
        }
```

The method is rather long, but keep in mind that roughly half of the code above  is error checking and logging. This is intended to be "Production-quality" code,  not just a minimal code sample.

### Adding reusable content to a page

Once you have the **Reusable Content **list item created (either  manually or programmatically using code like that shown above), the next task is  to add the content to a page. Since I'm assuming you already know how to do that  using the out-of-the-box page editing features in SharePoint, let's see how this  can be achieved through code.

First it is important to understand how reusable content is implemented in SharePoint  (the foundation is the same in MOSS 2007 and SharePoint 2010). There are two key  concepts to grasp:

1. The "storage format" of the content in a Publishing HTML field uses a special &lt;div&gt;
   element as a "header" and corresponding &lt;span&gt; elements to designate the
   placeholders where reusable content is to be inserted (note that I'm only talking
   about reusable content that is specified to automatically update).
2. The "view format" of the content is generated by reading the "header" and
   subsequently replacing the corresponding placeholders that follow. Hence I tend
   to refer to this as the "expanded" HTML. In *most* cases, you can get
   the expanded HTML using out-of-the-box SharePoint functionality. [In part 3
   of this series, I'll discuss when you can't get it that way and explain the
   not-so-elegant workaround that I came up with.]

Here is a sample of the HTML content in "storage format":

```
<div id="__publishingReusableFragmentIdSection">
        <a href="/ReusableContent/1_.000">a</a>
        <a href="/ReusableContent/3_.000">a</a>
    </div>
    <p>
        Here is some reusable content...</p>
    <p>
        <span id="__publishingReusableFragment"></span>
    </p>
    <p>
        ...and here is some more:</p>
    <p>
        <span id="__publishingReusableFragment"></span>
    </p>
```

The corresponding "view format" is shown below:

```
<p>
        Here is some reusable content...</p>
    <p>
        <span class="ms-rtestate-read  ms-reusableTextView"
            contenteditable="false" id="__publishingReusableFragment"
            fragmentid="/ReusableContent/1_.000">
            Copyright&copy; 2009 Contoso Corporation - All Rights
            Reserved</span>
    </p>
    <p>
        ...and here is some more:</p>
    <p>
        <span class="ms-rtestate-read  ms-reusableTextView"
            contenteditable="false" id="__publishingReusableFragment"
            fragmentid="/ReusableContent/3_.000">
            <em>&quot;Example quotation&quot;</em>
        </span>
    </p>
```

> **Note**
> 
> I have no idea why the out-of-the-box **Byline** and
> **Quote** reusable content items in SharePoint 2010 specify
> **Automatic Update** = **Yes**. I can see the
> reasoning for enabling automatic update of the default **Copyright
> **item, but these other two baffle me.

Look again at the sample "storage format" HTML above. Notice how the reusable  content placeholders (i.e. the &lt;span&gt; elements) do not specify which **Reusable Content **list item to render. Rather, the list item is  determined based on the position of the placeholder (relative to other placeholders)  -- which is matched to the corresponding item specified in the "header" (by index).

In other words, if you were to swap the order of the &lt;a&gt; elements in the  "header"...

```
<div id="__publishingReusableFragmentIdSection">
        <a href="/ReusableContent/3_.000">a</a>
        <a href="/ReusableContent/1_.000">a</a>
    </div>
    ...
```

...then the order of the reusable content in the corresponding "view format"  would be reversed, as shown below:

```
<p>
        Here is some reusable content...</p>
    <p>
        <span class="ms-rtestate-read  ms-reusableTextView"
            contenteditable="false" id="Span1"
            fragmentid="/ReusableContent/3_.000">
            <em>&quot;Example quotation&quot;</em>
        </span>
    </p>
    <p>
        ...and here is some more:</p>
    <p>
        <span class="ms-rtestate-read  ms-reusableTextView"
            contenteditable="false" id="__publishingReusableFragment"
            fragmentid="/ReusableContent/1_.000">
            Copyright&copy; 2009 Contoso Corporation - All Rights
            Reserved</span>
    </p>
```

This actually makes the code for inserting reusable content into Publishing HTML  fields significantly more complex than it would be if the "storage format" specified  something like the following instead:

```
<div id="__publishingReusableFragmentIdSection" />
    <p>
        Here is some reusable content...</p>
    <p>
        <span id="__publishingReusableFragment">
            <a href="/ReusableContent/1_.000">a</a>
        </span>
    </p>
    <p>
        ...and here is some more:</p>
    <p>
        <span id="__publishingReusableFragment">
            <a href="/ReusableContent/3_.000">a</a>
        </span>
    </p>
```

Rather than simply listing the code for inserting reusable content into a page  (which you can easily access in the attached sample solution), start by reviewing  some of the unit tests that I created when developing the **InsertReusableContentIntoHtmlField
**method:

```
/// <summary>
        /// Basic test for appending reusable content to an HTML field.
        /// </summary>
        [TestMethod()]
        public void InsertReusableContentIntoHtmlField001()
        {
            const string reusableContentListItemUrl = "/ReusableContent/1_.000";
            const string htmlFieldContent = null;

            const string expected =
                "<div id=\"__publishingReusableFragmentIdSection\">"
                        + "<a href=\"/ReusableContent/1_.000\">a</a>"
                    + "</div>"
                    + "<span id=\"__publishingReusableFragment\"></span>";

            string actual =
                SharePointHtmlFieldHelper.InsertReusableContentIntoHtmlField(
                    reusableContentListItemUrl,
                    htmlFieldContent);

            Assert.AreEqual(expected, actual);
        }

        /// <summary>
        /// Basic test for appending reusable content to an HTML field which
        /// already contains another piece of reusable content.
        /// </summary>
        [TestMethod()]
        public void InsertReusableContentIntoHtmlField002()
        {
            const string reusableContentListItemUrl = "/ReusableContent/2_.000";
            const string htmlFieldContent =
                "<div id=\"__publishingReusableFragmentIdSection\">"
                        + "<a href=\"/ReusableContent/1_.000\">a</a>"
                    + "</div>"
                    + "<span id=\"__publishingReusableFragment\"></span>";

            const string expected =
                "<div id=\"__publishingReusableFragmentIdSection\">"
                        + "<a href=\"/ReusableContent/1_.000\">a</a>"
                        + "<a href=\"/ReusableContent/2_.000\">a</a>"
                    + "</div>"
                    + "<span id=\"__publishingReusableFragment\"></span>"
                    + "<span id=\"__publishingReusableFragment\"></span>";

            string actual =
                SharePointHtmlFieldHelper.InsertReusableContentIntoHtmlField(
                    reusableContentListItemUrl,
                    htmlFieldContent);

            Assert.AreEqual(expected, actual);
        }

        /// <summary>
        /// Basic test for inserting reusable content into an HTML field at a
        /// specific location.
        /// </summary>
        [TestMethod()]
        public void InsertReusableContentIntoHtmlField003()
        {
            const string placeholder = "{TODO: Insert reusable content here}";

            const string reusableContentListItemUrl = "/ReusableContent/1_.000";
            const string htmlFieldContent = "<p>" + placeholder + "</p>";

            const string expected =
                "<div id=\"__publishingReusableFragmentIdSection\">"
                        + "<a href=\"/ReusableContent/1_.000\">a</a>"
                    + "</div>"
                    + "<p><span id=\"__publishingReusableFragment\"></span></p>";

            string actual =
                SharePointHtmlFieldHelper.InsertReusableContentIntoHtmlField(
                    reusableContentListItemUrl,
                    htmlFieldContent,
                    placeholder);

            Assert.AreEqual(expected, actual);
        }

        /// <summary>
        /// Basic test for inserting reusable content into an HTML field which
        /// already contains other pieces of reusable content.
        /// </summary>
        [TestMethod()]
        public void InsertReusableContentIntoHtmlField004()
        {
            const string placeholder = "{TODO: Insert reusable content here}";

            const string reusableContentListItemUrl = "/ReusableContent/3_.000";
            const string htmlFieldContent =
                "<div id=\"__publishingReusableFragmentIdSection\">"
                        + "<a href=\"/ReusableContent/1_.000\">a</a>"
                        + "<a href=\"/ReusableContent/2_.000\">a</a>"
                    + "</div>"
                    + "<div id='reusableContent1'>"
                        + "<span id=\"__publishingReusableFragment\"></span>"
                    + "</div>"
                    + "<div id='reusableContent3'>" + placeholder + "</div>"
                    + "<div id='reusableContent2'>"
                        + "<span id=\"__publishingReusableFragment\"></span>"
                    + "</div>";

            const string expected =
                "<div id=\"__publishingReusableFragmentIdSection\">"
                        + "<a href=\"/ReusableContent/1_.000\">a</a>"
                        + "<a href=\"/ReusableContent/3_.000\">a</a>"
                        + "<a href=\"/ReusableContent/2_.000\">a</a>"
                    + "</div>"
                    + "<div id='reusableContent1'>"
                        + "<span id=\"__publishingReusableFragment\"></span>"
                    + "</div>"
                    + "<div id='reusableContent3'>"
                        + "<span id=\"__publishingReusableFragment\"></span>"
                    + "</div>"
                    + "<div id='reusableContent2'>"
                        + "<span id=\"__publishingReusableFragment\"></span>"
                    + "</div>";

            string actual =
                SharePointHtmlFieldHelper.InsertReusableContentIntoHtmlField(
                    reusableContentListItemUrl,
                    htmlFieldContent,
                    placeholder);

            Assert.AreEqual(expected, actual);
        }
```

> **Note**
> 
> Initially, I added the code for inserting reusable content into a page to
> the **SharePointPublishingHelper **class. However, I ended
> up refactoring this code into the new **SharePointHtmlFieldHelper
> **class. I'll discuss this new class in more detail in part 3 of
> this series.

### Sample "Reusable Content" solution for SharePoint Server 2010

I've attached a complete Visual Studio 2010 solution so you can see just how  easy it is to automatically create reusable content and add it to a page using the  provided helper classes (for example, upon activation of a feature).

If you've deployed any of my other sample SharePoint solutions, you'll find this  one just as easy.

Here are the instructions to deploy the sample to your own SharePoint environment.  First, download the attachment and unzip the files. Then you simply need to create  a few domain users and run a handful of PowerShell scripts, as described below.

#### To deploy the sample solution to SharePoint 2010:

1. Create three service accounts for the Fabrikam Demo site:
   
   - **{DOMAIN}\svc-web-fabrikam-dev** - used as the application
     pool identity for the new "Fabrikam Demo" site
   - **{DOMAIN}\svc-sp-psr-dev** - object cache user account
     providing Full Read access to Web applications ([http://technet.microsoft.com/en-us/library/ff758656.aspx](http://technet.microsoft.com/en-us/library/ff758656.aspx))
   - **{DOMAIN}\svc-sp-psu-dev** - object cache user account
     providing Full Control access to Web applications

2. On the **Start** menu, click **All Programs**,
   click **Microsoft SharePoint 2010 Products**, right-click
   **SharePoint 2010 Management Shell**, and then click **Run
   as administrator**. If prompted by User Account Control to allow the
   program to make changes to the computer, click **Yes**.

3. From the Windows PowerShell command prompt, change to the directory containing
   the deployment scripts (e.g. C:\NotBackedUp\Fabrikam\Demo\Dev\SharePointReusableContent\Source\DeploymentFiles\Scripts),
   and run the following commands:
   
   ```
   $env:FABRIKAM_DEMO_URL = "http://fabrikam-local"
   ```
   
   ```
   $env:FABRIKAM_DEMO_BUILD_CONFIGURATION = "Debug"
   ```
   
   ```
   & '.\Add Event Log Sources.ps1'
   ```
   
   ```
   & '.\Create Web Application.ps1'
   ```
   
   ```
   & '.\Configure Object Cache User Accounts.ps1'
   ```
   
   ```
   & '.\Create Site Collections.ps1'
   ```
   
   ```
   & '.\Enable Anonymous Access.ps1'
   ```
   
   ```
   & '.\Add Solutions.ps1'
   ```
   
   ```
   & '.\Deploy Solutions.ps1'
   ```
   
   ```
   & '.\Activate Features.ps1'
   ```

> **Note**
> 
> Technically, you don't have to set the environment variables (and use the
> "-dev" accounts). However, I recommend this in order to bypass SharePoint
> timer jobs when deploying the WSPs.

At this point you should be able to modify your hosts file accordingly and browse  to either [http://www-local.fabrikam.com](http://www-local.tugboatcoffee.com)  (to view the site as an anonymous user) or [http://fabrikam-local](http://tugboatcoffee-local) (to view the site as an administrator).

You can then click the **Reusable Content Sample **link on the home  page of the site to view the sample page. Once you have verified the page renders  as expected, modify the corresponding item in the **Reusable Content
**list and verify the page is "automatically updated" accordingly. [Now that  you understand how reusable content works in SharePoint, you know the page isn't  "automatically updated" at all. Rather, the dynamically generated "view format"  renders the updated content.]

In [part 3 of this series](/blog/jjameson/2011/04/14/reusable-content-in-sharepoint-publishing-html-fields-part-3), I'll discuss various ways of accessing the "expanded"  HTML content (a.k.a. the "view format.")

