---
title: "Introducing the SharePointPublishingHelper Class"
date: 2009-10-09T07:19:00-06:00
excerpt: "In my previous post , I described a utility to import pages into Microsoft Office SharePoint Server (MOSS) 2007 from an Excel input file. 
 Aside from the code to read data from the input file into a DataSet, the main work is performed by the SharePointPublishingHelper..."
aliases: ["/blog/jjameson/archive/2009/10/08/introducing-the-sharepointpublishinghelper-class.aspx", "/blog/jjameson/archive/2009/10/09/introducing-the-sharepointpublishinghelper-class.aspx"]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/09/introducing-the-sharepointpublishinghelper-class.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/09/introducing-the-sharepointpublishinghelper-class.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

In my [previous post](/blog/jjameson/2009/10/08/importing-pages-into-moss-2007-from-an-excel-file), I described a utility to import pages into Microsoft Office SharePoint Server (MOSS) 2007 from an Excel input file.

Aside from the code to read data from the input file into a DataSet, the main work is performed by the `SharePointPublishingHelper` class (which was written long before the ImportPages.exe utility). This class is simply a collection of helper methods for the SharePoint publishing API.

`SharePointPublishingHelper` makes it really easy to:

- Attempt to retrieve a reference to a [PublishingPage](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.publishing.publishingpage.aspx) by specifying an [SPWeb](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.spweb.aspx) and the page URL name (e.g. default.aspx)
- Ensure an expected page exists in a site (creating a new page as necessary)
- Check if a page is checked out to anyone
- Check if a page is checked out by the current user
- Ensure a page is checked out to the current user (since you can't make any changes to a page that isn't checked out to you)
- Ensure a page is associated with a specific page layout (and corresponding content type)
- Publish a page (correctly handling page libraries that are configured for approval as well as those that are not configured for approval)
- Delete a page by specifying the page URL name

Here's the code for the `SharePointPublishingHelper` class:

```
using System;
using System.Diagnostics;
using System.Globalization;
using System.Security.Permissions;

using Microsoft.SharePoint;
using Microsoft.SharePoint.Publishing;
using Microsoft.SharePoint.Security;

using Fabrikam.Demo.CoreServices.Logging;

namespace Fabrikam.Demo.CoreServices.SharePoint
{
    /// <summary>
    /// Exposes static methods for commonly used helper functions for the
    /// SharePoint Publishing API. This class cannot be inherited.
    /// </summary>
    /// <remarks>
    /// All methods of the <b>SharePointPublishingHelper</b> class are static
    /// and can therefore be called without creating an instance of the class.
    /// </remarks>
    [CLSCompliant(false)]
    [SharePointPermission(SecurityAction.LinkDemand, ObjectModel = true)]
    public sealed class SharePointPublishingHelper
    {
        private SharePointPublishingHelper() { } // all members are static

        /// <summary>
        /// Deletes the specified publishing page in the site (if it exists).
        /// </summary>
        /// <param name="web">The site containing the publishing page.</param>
        /// <param name="pageUrlName">The URL name of the page (e.g.
        /// "default.aspx").</param>
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Design",
            "CA1054:UriParametersShouldNotBeStrings",
            Justification = "Page URL name is only the last part of the local"
                + " path of the page URL.")]
        public static void DeletePage(
            SPWeb web,
            string pageUrlName)
        {
            if (web == null)
            {
                throw new ArgumentNullException("web");
            }
            else if (string.IsNullOrEmpty(pageUrlName) == true)
            {
                throw new ArgumentException(
                    "The page URL name must be specified.",
                    "pageUrlName");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Deleting page ({0}) from Web ({1})...",
                pageUrlName,
                web.Url);

            PublishingPage page = FindPublishingPage(web, pageUrlName);
            
            if (page == null)
            {
                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "The page specified to delete ({0})"
                        + " does not exist in the Web ({1}).",
                    pageUrlName,
                    web.Url);

            }
            else
            {
                string absolutePageUrl =
                        page.ListItem.Web.Url + '/' + page.Url;
                
                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "Deleting page: {0}...",
                    absolutePageUrl);

                page.ListItem.Delete();

                Logger.LogInfo(
                    CultureInfo.InvariantCulture,
                    "Successfully deleted page: {0}.",
                    absolutePageUrl);
            }
        }

        /// <summary>
        /// Ensures a page exists and is configured according with the
        /// specified settings.
        /// </summary>
        /// <param name="web">The site containing the publishing page.</param>
        /// <param name="pageUrlName">The URL name of the page (e.g.
        /// "default.aspx").</param>
        /// <param name="pageTitle">The title of the page.</param>
        /// <param name="pageLayoutUrl">The URL of the page layout for the
        /// page (e.g. /_catalogs/masterpage/WelcomeLinks.aspx).</param>
        /// <returns>The new or existing publishing page.</returns>
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Design",
            "CA1054:UriParametersShouldNotBeStrings",
            Justification = "Page URL name is only the last part of the local"
                + " path of the page URL. Page layout URL is a server-relative"
                + " URL (e.g. /_catalogs/masterpage/WelcomeLinks.aspx).")]
        public static PublishingPage EnsurePage(
            SPWeb web,
            string pageUrlName,
            string pageTitle,
            string pageLayoutUrl)
        {
            return EnsurePage(
                web,
                pageUrlName,
                pageTitle,
                pageLayoutUrl,
                null);
        }

        /// <summary>
        /// Ensures a page exists and is configured according with the
        /// specified settings.
        /// </summary>
        /// <param name="web">The site containing the publishing page.</param>
        /// <param name="pageUrlName">The URL name of the page (e.g.
        /// "default.aspx").</param>
        /// <param name="pageTitle">The title of the page.</param>
        /// <param name="pageLayoutUrl">The URL of the page layout for the
        /// page (e.g. /_catalogs/masterpage/WelcomeLinks.aspx).</param>
        /// <param name="pageDescription">The description of the page (optional).</param>
        /// <returns>The new or existing publishing page.</returns>
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Design",
            "CA1054:UriParametersShouldNotBeStrings",
            Justification = "Page URL name is only the last part of the local"
                + " path of the page URL. Page layout URL is a server-relative"
                + " URL (e.g. /_catalogs/masterpage/WelcomeLinks.aspx).")]
        public static PublishingPage EnsurePage(
            SPWeb web,
            string pageUrlName,
            string pageTitle,
            string pageLayoutUrl,
            string pageDescription)
        {
            if (web == null)
            {
                throw new ArgumentNullException("web");
            }
            else if (pageUrlName == null)
            {
                throw new ArgumentNullException("pageUrlName");
            }
            else if (string.IsNullOrEmpty(pageUrlName) == true)
            {
                throw new ArgumentException(
                    "The page URL name must be specified.",
                    "pageUrlName");
            }
            else if (pageTitle == null)
            {
                throw new ArgumentNullException("pageTitle");
            }
            else if (string.IsNullOrEmpty(pageTitle) == true)
            {
                throw new ArgumentException(
                    "The page title must be specified.",
                    "pageTitle");
            }
            else if (pageLayoutUrl == null)
            {
                throw new ArgumentNullException("pageLayoutUrl");
            }
            else if (string.IsNullOrEmpty(pageLayoutUrl) == true)
            {
                throw new ArgumentException(
                    "The page layout URL must be specified.",
                    "pageLayoutUrl");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Ensuring page ({0}) in Web ({1})...",
                pageUrlName,
                web.Url);

            PublishingSite pubSite = new PublishingSite(web.Site);
            PageLayout layout = pubSite.PageLayouts[pageLayoutUrl];

            if (layout == null)
            {
                string message = string.Format(
                    CultureInfo.InvariantCulture,
                    "Invalid page layout specified ({0}).",
                    pageLayoutUrl);

                throw new ArgumentException(
                    message,
                    "pageLayoutUrl");
            }

            PublishingWeb pubWeb = PublishingWeb.GetPublishingWeb(web);
            PublishingPageCollection pages = pubWeb.GetPublishingPages();

            PublishingPage page =
                pages[pubWeb.PagesListName + "/" + pageUrlName];

            if (page == null)
            {
                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "Creating page ({0}/{1}/{2})...",
                    web.Url,
                    pubWeb.PagesListName,
                    pageUrlName);

                page = pages.Add(pageUrlName, layout);
                page.Title = pageTitle;

                if (string.IsNullOrEmpty(pageDescription) == false)
                {
                    page.Description = pageDescription;
                }

                page.ListItem.SystemUpdate(false);

                Logger.LogInfo(
                    CultureInfo.InvariantCulture,
                    "Successfully created page: {0}/{1}.",
                    web.Url,
                    page.Url);
            }
            else
            {
                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "The specified page ({0}/{1}) already exists.",
                    web.Url,
                    page.Url);
            }

            return page;
        }

        /// <summary>
        /// Ensure the specified page is configured with the specified content
        /// type and page layout.
        /// </summary>
        /// <param name="page">The page to check (and update as necessary).</param>
        /// <param name="pageLayoutUrl">The URL of the page layout for the
        /// page (e.g. /_catalogs/masterpage/WelcomeLinks.aspx).</param>
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Design",
            "CA1054:UriParametersShouldNotBeStrings",
            Justification = "Page layout URL is a server-relative"
                + " URL (e.g. /_catalogs/masterpage/WelcomeLinks.aspx).")]
        public static void EnsurePageContentTypeAndLayout(
            PublishingPage page,
            string pageLayoutUrl)
        {
            if (page == null)
            {
                throw new ArgumentNullException("page");
            }

            if (string.IsNullOrEmpty(pageLayoutUrl) == true)
            {
                throw new ArgumentException(
                    "The page layout must be specified.",
                    "pageLayoutUrl");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Ensuring content type and page layout ({0}) for page"
                    + " ({1}/{2})...",
                pageLayoutUrl,
                page.ListItem.Web.Url,
                page.Url);

            PublishingSite site = new PublishingSite(page.PublishingWeb.Web.Site);
            PageLayout layout = site.PageLayouts[pageLayoutUrl];
            
            SPContentTypeId contentTypeId = layout.AssociatedContentType.Id;
            
            // HACK: The content type associated with each "Pages" library
            // actually gets a unique ContentTypeId (it is appended with a
            // unique identifier for the list -- thus inheriting from the
            // content type associated with the page layout).
            SPContentType pageContentType = null;
            
            foreach (SPContentType listContentType in
                page.PublishingWeb.PagesList.ContentTypes)
            {
                if (listContentType.Parent.Id == contentTypeId)
                {
                    pageContentType = listContentType;
                    break;
                }
            }
            
            if (pageContentType == null)
            {
                string message = string.Format(
                    CultureInfo.InvariantCulture,
                    "The content type ({0}) associated with the page layout"
                        + " ({1}) is not enabled on the list ({2}/{3}).",
                    layout.AssociatedContentType.Name,
                    pageLayoutUrl,
                    page.ListItem.Web.Url,
                    page.PublishingWeb.PagesListName);

                throw new ArgumentException(message, "pageLayoutUrl");
            }

            if ((SPContentTypeId)page.ListItem["ContentTypeId"] !=
                    pageContentType.Id
                || page.Layout.Name != layout.Name)
            {
                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "Updating content type and page layout ({0}) for page"
                        + " ({1}/{2})...",
                    pageLayoutUrl,
                    page.ListItem.Web.Url,
                    page.Url);

                page.ListItem["ContentTypeId"] = pageContentType.Id;
                page.Layout = layout;
                page.Update();

                Logger.LogInfo(
                    CultureInfo.InvariantCulture,
                    "Successfully updated content type and page layout ({0})"
                        + "  for page: {1}/{2}.",
                    pageLayoutUrl,
                    page.ListItem.Web.Url,
                    page.Url);
            }
            else
            {
                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "The page ({0}/{1}) is already configured with the"
                        + " specified content type and page layout ({2}).",
                    page.ListItem.Web.Url,
                    page.Url,                    
                    pageLayoutUrl);
            }
        }

        /// <summary>
        /// Ensures the specified page is checked out to the current user.
        /// </summary>
        /// <param name="page">The publishing page to check out.</param>
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Naming",
            "CA1702:CompoundWordsShouldBeCasedCorrectly",
            MessageId = "ToMe")]
        public static void EnsurePageIsCheckedOutToMe(
            PublishingPage page)
        {
            if (page == null)
            {
                throw new ArgumentNullException("page");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Ensuring page ({0}/{1}) is checked out to the current"
                    + " user...",
                page.ListItem.Web.Url,
                page.Url);

            bool isPageCheckedOut = IsPageCheckedOut(page);

            if (isPageCheckedOut == true)
            {
                bool isPageCheckedOutToMe = IsPageCheckedOutToMe(page);

                if (isPageCheckedOutToMe == true)
                {
                    Logger.LogDebug(
                        CultureInfo.InvariantCulture,
                        "The page ({0}/{1}) is already checked out to the"
                            + " current user.",
                        page.ListItem.Web.Url,
                        page.Url);

                    return;
                }

                string message = string.Format(
                    CultureInfo.InvariantCulture,
                    "Cannot check out the page "
                        + " ({0}/{1}) because it is already checked"
                        + " out to somebody else.",
                    page.ListItem.Web.Url,
                    page.Url);

                throw new InvalidOperationException(message);
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Checking out page ({0}/{1})...",
                page.ListItem.Web.Url,
                page.Url);

            page.CheckOut();

            Logger.LogInfo(
                CultureInfo.InvariantCulture,
                "Successfully checked out page: {0}/{1}.",
                page.ListItem.Web.Url,
                page.Url);
        }

        /// <summary>
        /// Finds the specified publishing page in the site.
        /// </summary>
        /// <param name="web">The site containing the publishing page.</param>
        /// <param name="pageUrlName">The URL name of the page (e.g.
        /// "default.aspx").</param>
        /// <returns>The specified publishing page, or null if the page does not
        /// exist.</returns>
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Design",
            "CA1054:UriParametersShouldNotBeStrings",
            Justification = "Page URL name is only the last part of the local"
                + " path of the page URL.")]
        public static PublishingPage FindPublishingPage(
            SPWeb web,
            string pageUrlName)
        {
            if (web == null)
            {
                throw new ArgumentNullException("web");
            }
            else if (string.IsNullOrEmpty(pageUrlName) == true)
            {
                throw new ArgumentException(
                    "The page URL name must be specified.",
                    "pageUrlName");
            }

            PublishingWeb pubWeb = PublishingWeb.GetPublishingWeb(web);
            PublishingPageCollection pages = pubWeb.GetPublishingPages();

            PublishingPage page = pages[pubWeb.PagesListName + "/" + pageUrlName];
            return page;
        }

        /// <summary>
        /// Determines if the specified page is approved.
        /// </summary>
        /// <param name="page">The page to determine the status of.</param>
        /// <returns>True if the specified page is approved (or if the
        /// corresponding library is not configured for approval), otherwise
        /// false.</returns>
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Design",
            "CA1011:ConsiderPassingBaseTypesAsParameters")]
        public static bool IsPageApproved(
            PublishingPage page)
        {
            if (page == null)
            {
                throw new ArgumentNullException("page");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Checking if page ({0}/{1}) is approved...",
                page.ListItem.Web.Url,
                page.Url);

            bool isApproved = false;

            if (page.ListItem.ModerationInformation == null)
            {
                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "Pages library ({0}/{1}) is not configured for approval.",
                    page.ListItem.Web.Url,
                    page.PublishingWeb.PagesListName);

                Debug.Assert(page.ListItem.Versions != null);
                Debug.Assert(page.ListItem.Versions.Count > 0);

                if (page.ListItem.Versions[0].Level == SPFileLevel.Published)
                {
                    isApproved = true;
                }
            }
            else
            {

                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "Pages library ({0}/{1}) is configured for approval.",
                    page.ListItem.Web.Url,
                    page.PublishingWeb.PagesListName);

                isApproved =
                    (page.ListItem.ModerationInformation.Status ==
                        SPModerationStatusType.Approved);
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "The page ({0}/{1} is {2}.",
                page.ListItem.Web.Url,
                page.Url,
                isApproved ? "approved" : "not approved");

            return isApproved;
        }

        /// <summary>
        /// Determines if the specified page is checked out.
        /// </summary>
        /// <param name="page">The page to determine the status of.</param>
        /// <returns>True if the specified page is checked out, otherwise false.</returns>
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Design",
            "CA1011:ConsiderPassingBaseTypesAsParameters")]
        public static bool IsPageCheckedOut(
            PublishingPage page)
        {
            if (page == null)
            {
                throw new ArgumentNullException("page");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Checking if page ({0}/{1}) is checked out...",
                page.ListItem.Web.Url,
                page.Url);

            bool isCheckedOut =
                (page.ListItem.File.CheckOutStatus != SPFile.SPCheckOutStatus.None);

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "The page ({0}/{1}) is {2}.",
                page.ListItem.Web.Url,
                page.Url,
                isCheckedOut ? "checked out" : "not checked out");

            return isCheckedOut;
        }

        /// <summary>
        /// Determines if the specified page is checked out to the current user.
        /// </summary>
        /// <param name="page">The page to determine the status of.</param>
        /// <returns>True if the specified page is checked out to the current
        /// user, otherwise false.</returns>
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Design",
            "CA1011:ConsiderPassingBaseTypesAsParameters"),
        System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Naming",
            "CA1702:CompoundWordsShouldBeCasedCorrectly",
            MessageId = "ToMe")]
        public static bool IsPageCheckedOutToMe(
            PublishingPage page)
        {
            if (page == null)
            {
                throw new ArgumentNullException("page");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Checking if page ({0}/{1}) is checked out to current user...",
                page.ListItem.Web.Url,
                page.Url);

            bool isCheckedOut = (page.ListItem.File.CheckOutStatus
                != SPFile.SPCheckOutStatus.None);

            if (isCheckedOut == false)
            {
                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "The page ({0}/{1}) is not checked out.",
                    page.ListItem.Web.Url,
                    page.Url);

                return false;
            }

            bool isCheckedOutToMe = (page.ListItem.File.CheckedOutBy.LoginName
                == page.ListItem.Web.CurrentUser.LoginName);

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "The page ({0}/{1}) is {2} to the current user.",
                page.ListItem.Web.Url,
                page.Url,
                isCheckedOutToMe ? "checked out" : "not checked out");

            return isCheckedOutToMe;
        }

        /// <summary>
        /// Publishes the specified page (making it generally available for viewing).
        /// </summary>
        /// <param name="page">The page to publish.</param>
        /// <param name="comments">Comments to associate with the publishing action.</param>
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Design",
            "CA1011:ConsiderPassingBaseTypesAsParameters")]
        public static void PublishPage(
            PublishingPage page,
            string comments)
        {
            if (page == null)
            {
                throw new ArgumentNullException("page");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Publishing page ({0}/{1})...",
                page.ListItem.Web.Url,
                page.Url);

            if (page.ListItem.ModerationInformation == null)
            {
                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "Pages library ({0}/{1}) is not configured for approval."
                        + " Checking in page...",
                    page.ListItem.Web.Url,
                    page.PublishingWeb.PagesListName);

                page.ListItem.File.CheckIn(
                    comments,
                    SPCheckinType.MajorCheckIn);
            }
            else
            {
                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "Pages library ({0}/{1}) is configured for approval."
                        + " Scheduling page for publication...",
                    page.ListItem.Web.Url,
                    page.PublishingWeb.PagesListName);

                page.Schedule(comments);
            }

            Logger.LogInfo(
                CultureInfo.InvariantCulture,
                "Successfully published page: {0}/{1}.",
                page.ListItem.Web.Url,
                page.Url);
        }
    }
}
```

In addition to the `SharePointPublishingHelper` class, I often rely on several other custom SharePoint "helper" classes, such `SharePointListHelper` and `SharePointWebPartHelper`. These classes evolved over time by refactoring common code that was previously repeated in numerous places. In other words, as Scott Hanselman likes to say...a developer should always try to write DRY code (meaning, don't repeat yourself).

