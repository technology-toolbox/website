---
title: "Inserting Web Parts into Publishing Fields in SharePoint 2010"
date: 2011-03-02T01:25:00-07:00
excerpt: "In the sample SharePoint solution I provided in one of last week's posts , you may have noticed that when programmatically creating the custom Sign In page, I insert the custom Claims Login Form Web Part into the Page Content field. 
 In Microsoft Office..."
aliases: ["/blog/jjameson/archive/2011/03/02/inserting-web-parts-into-publishing-fields-in-sharepoint-2010.aspx"]
draft: true
categories: ["SharePoint"]
tags: ["SharePoint 2010"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/03/02/inserting-web-parts-into-publishing-fields-in-sharepoint-2010.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/03/02/inserting-web-parts-into-publishing-fields-in-sharepoint-2010.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

In the sample SharePoint solution I provided in [one of last week's posts](/blog/jjameson/2011/02/25/claims-login-web-part-for-sharepoint-server-2010), you may have noticed that when programmatically creating the custom **Sign In** page, I insert the custom Claims Login Form Web Part into the **Page Content** field.

In Microsoft Office SharePoint Server (MOSS) 2007, you could only add Web Parts to predetermined zones on the page. However, in SharePoint 2010, you can add a Web Part directly to page content fields. This is huge, because inserting Web Parts "inline" anywhere you want within the main content of the page greatly reduces the number of page layouts that you need to create (simply to achieve different combinations of static HTML content and dynamic content rendered by one or more Web Parts).

The "trick" to rendering a Web Part inline with other content is to add the Web Part to a special "hidden" zone ("wpz") and then insert a "placeholder" &lt;div&gt; element where you want the Web Part to appear within the HTML content. The following post does a great job of explaining what's happening behind the scenes when you insert a Web Part into the "Rich Content" content field on a page:

{{< reference title="Programmatically adding Web Parts to Rich Content in SharePoint 2010" linkHref="http://blog.mastykarz.nl/programmatically-adding-web-parts-rich-content-sharepoint-2010/" >}}

Long ago, when I first started programmatically creating Publishing pages in MOSS 2007, I ended up creating a **[SharePointPublishingHelper](/blog/jjameson/2009/10/09/introducing-the-sharepointpublishinghelper-class)** class. For those of you that haven't seen the original post (and don't want to take the time to read it now), you just need to understand that **SharePointPublishingHelper** is simply intended to make the process of creating, configuring, and approving Publishing pages in SharePoint as "painless" as possible.

Consequently, for SharePoint Server 2010, I added the following method to **SharePointPublishingHelper**:

```
public static void InsertWebPartIntoPageContent(
            Part webPart,
            PublishingPage page,
            Guid fieldId,
            string placeholder)
        {
            if (webPart == null)
            {
                throw new ArgumentNullException("webPart");
            }            
            else if (page == null)
            {
                throw new ArgumentNullException("page");
            }
            else if (fieldId == Guid.Empty)
            {
                throw new ArgumentException(
                    "The field ID must be specified.",
                    "fieldId");
            }

            string webPartName = string.IsNullOrEmpty(webPart.Title) ?
                webPart.GetType().Name : webPart.Title;

            string fieldContent = (string)page.ListItem[fieldId];

            if (string.IsNullOrEmpty(placeholder) == false
                && (string.IsNullOrEmpty(fieldContent) == true
                    || fieldContent.Contains(placeholder) == false))
            {
                string message = string.Format(
                    CultureInfo.CurrentCulture,
                    "The field on the page ({0}/{1}) does not contain the"
                        + " expected placeholder ({2}) for the Web Part ({3}).",
                    page.ListItem.Web.Url,
                    page.Url,
                    placeholder,
                    webPartName);

                throw new InvalidOperationException(message);
            }
            
            if (string.IsNullOrEmpty(webPart.ID) == true
                || webPart.ID.StartsWith(
                    "g_",
                    StringComparison.OrdinalIgnoreCase) == false)
            {
                throw new ArgumentException(
                    "The Web Part ID is expected to begin with \"g_\".",
                    "webPart");

            }

            SPLogger.Log(
                LogCategory.Configuration,
                TraceSeverity.High,
                "Inserting Web Part ({0}) into content on page ({1}/{2})...",
                webPartName,
                page.ListItem.Web.Url,
                page.Url);

            string temp = webPart.ID.Substring("g_".Length);
            temp = temp.Replace('_', '-');

            Guid storageKey = new Guid(temp);

            string webPartMarker = string.Format(
                CultureInfo.InvariantCulture,
                "<div class=\"ms-rtestate-read ms-rte-wpbox\""
                    + " contentEditable=\"false\">"
                        + "<div class=\"ms-rtestate-read {0}\" id=\"div_{0}\">"
                        +"</div>"
                        + "<div style='display:none' id=\"vid_{0}\">"
                        + "</div>"
                    + "</div>",
                storageKey.ToString("D"));

            if (string.IsNullOrEmpty(placeholder) == false)
            {
                SPLogger.Log(
                    LogCategory.Configuration,
                    TraceSeverity.Verbose,
                    "Replacing placeholder with Web Part ({0}) on page"
                        + "({1}/{2})...",
                    webPartName,
                    page.ListItem.Web.Url,
                    page.Url);

                page.ListItem[fieldId] = fieldContent.Replace(
                    placeholder,
                    webPartMarker);
            }
            else
            {
                SPLogger.Log(
                    LogCategory.Configuration,
                    TraceSeverity.Verbose,
                    "Appending Web Part ({0}) to content on page"
                        + "({1}/{2})...",
                    webPartName,
                    page.ListItem.Web.Url,
                    page.Url);

                page.ListItem[fieldId] = fieldContent + webPartMarker;
            }

            page.Update();

            SPLogger.LogEvent(
                LogCategory.Configuration,
                EventSeverity.Information,
                "Successfully inserted Web Part ({0}) into content on page"
                    + " ({1}/{2}).",
                webPartName,
                page.ListItem.Web.Url,
                page.Url);
        }
```

If you rip out all of the error handling and logging, you'll see this new method is essentially the same as the code sample provided by Waldek in the post referenced above.

Note that I also added an overload for the **InsertWebPartIntoPageContent** method that simply appends the Web Part to the end of the field (rather than replacing some arbitrary placeholder text):

```
public static void InsertWebPartIntoPageContent(
            Part webPart,
            PublishingPage page,
            Guid fieldId)
        {
            InsertWebPartIntoPageContent(
                webPart,
                page,
                fieldId,
                null);
        }
```

In order to create a new page and add a Web Part at a specific location within the page content, all I need to do is use a little bit of code to create the page, set the default page content, and subsequently replace the placeholder in the page content with the desired Web Part:

```
private const string loginFormPlaceholder =
            "{TODO: Insert Claims Login Form Web Part here}";

        // TODO: Replace embedded CSS layout with custom SharePoint page layout
        private const string defaultSignInPageContent =
@"<div class='container_12' style='margin: auto; width: 800px'><div class='loginForm'
        style='width: 240px; display: inline; float: left; margin-right: 10px'>
" + loginFormPlaceholder + @"</div>
</div>
<div class='termsAndConditions'
    style='width: 460px; display: inline; float: left; margin-right: 10px'><h2>Terms and Conditions</h2><p class='ms-rteElement-P'>Lorem ipsum dolor sit amet...</p>
</div>";

        private static void ConfigureSignInPage(
            SPWeb web)
        {
            Debug.Assert(web != null);

            PublishingPage page =
                SharePointPublishingHelper.EnsurePage(
                    web,
                    "Sign-In.aspx",
                    "Sign In",
                    bodyOnlyLayoutUrl);

            ...		
            SharePointPublishingHelper.SetDefaultPageContent(
                page,
                defaultSignInPageContent);

            // Configure Web Parts
            SPWebPartPages.SPLimitedWebPartManager wpm =
                web.GetLimitedWebPartManager(
                    page.Url,
                    PersonalizationScope.Shared);

            using (wpm)
            {
                ReplacePlaceholderWithLoginWebPart(page, wpm);

                ...
            }

            SharePointPublishingHelper.PublishPage(
                page,
                "Published by Fabrikam.Demo.Web_HomeSiteConfiguration"
                    + " feature.");
        }
```

> **Note**
>
> The embedded CSS styles shown in the default page content above (e.g. margins, floats, and widths) are only intended for demonstration purposes. While I would have preferred to use something like the 960 Grid System instead, I was trying to minimize complexity in the original sample for the Claims Login Form Web Part.

Note that the **ReplacePlaceholderWithLoginWebPart** method simply checks to see if the Web Part placeholder is found in the page content and, if it is, subsequently uses the **SharePointPublishingHelper.InsertWebPartIntoPageContent** method to replace it with an instance of the Web Part:

```
private static void ReplacePlaceholderWithLoginWebPart(
            PublishingPage page,
            SPWebPartPages.SPLimitedWebPartManager wpm)
        {
            string pageContent =
                (string)page.ListItem[FieldId.PublishingPageContent];

            if (pageContent.Contains(loginFormPlaceholder) == true)
            {
                using (WebPart loginForm = new ClaimsLoginFormWebPart())
                {
                    loginForm.Title = "Claims Login Form";
                    loginForm.ChromeType = PartChromeType.None;

                    wpm.AddWebPart(
                        loginForm,
                        SharePointPublishingHelper.PageContentWebPartZone,
                        0);

                    SharePointPublishingHelper.InsertWebPartIntoPageContent(
                        loginForm,
                        page,
                        FieldId.PublishingPageContent,
                        loginFormPlaceholder);
                }
            }
        }
```

You might be wondering why I check to see if the placeholder exists in the content, rather than just assuming it will be there since it's obviously embedded in the default content specified in the code. The reason is because the Sign In page might have been modified by a content author after it was originally created and configured during activation of the feature. (This also explains the "set default content" logic as well as the code to skip the configuration of the page if it is already approved -- because we don't want to trample any customization to the page made after it was initially created.)

