---
title: Introducing the SharePointWebPartHelper Class
date: 2009-10-17T04:54:00-06:00
excerpt:
  In a previous post, I introduced the DR.DADA approach to SharePoint
  development and how I typically use the concept of a FeatureConfigurator to
  automatically configure one or more aspects of a SharePoint site when
  activating my feature. 
   For example...
aliases:
  [
    "/blog/jjameson/archive/2009/10/16/introducing-the-sharepointwebparthelper-class.aspx",
    "/blog/jjameson/archive/2009/10/17/introducing-the-sharepointwebparthelper-class.aspx",
  ]
draft: true
categories: ["My System", "SharePoint"]
tags: ["My System", "MOSS 2007", "WSS v3"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/17/introducing-the-sharepointwebparthelper-class.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/17/introducing-the-sharepointwebparthelper-class.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

In a previous post, I introduced the
[DR.DADA approach to SharePoint development](/blog/jjameson/2009/03/31/introducing-the-dr-dada-approach-to-sharepoint-development)
and how I typically use the concept of a
[FeatureConfigurator](/blog/jjameson/2007/03/22/what-s-in-a-name-defaultfeaturereceiver-vs-featureconfigurator)
to automatically configure one or more aspects of a SharePoint site when
activating my feature.

For example, on my latest project, we needed a login page as well as a legal
disclaimer page for a customer service portal based on Microsoft Office
SharePoint Server (MOSS) 2007. Thus I created a "PublicSiteConfiguration"
feature that, upon activation:

1. Automatically creates the **/Public** site
2. Populates the page content on the /Public/Pages/default.aspx page and adds an
   instance of a custom Login Form Web Part to the page
3. Creates the /Public/Pages/Disclaimer.aspx page and populates its default
   content

I've already introduced the `SharePointPublishingHelper` class that performs the
bulk of the work in creating and configuring publishing pages. In this post, I
want to cover the `SharePointWebPartHelper` class. Like the
`SharePointPublishingHelper` class, the `SharePointWebPartHelper` class is
simply a collection of helper methods for the SharePoint API.

`SharePointWebPartHelper` makes it really easy to:

- Attempt to retrieve a reference to a Web Part by specifying an
  [SPLimitedWebPartManager](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.webpartpages.splimitedwebpartmanager.aspx)
  for a specific page and the title of the Web Part you are trying to find
- Ensure a Web Part exists on a page (creating a new Web Part as necessary)
- Ensuring a
  [ContentEditorWebPart](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.webpartpages.contenteditorwebpart.aspx)
  exists on a page and that it is configured with the specified HTML content and
  title (as well as removing the "chrome", since -- at least in my experience --
  you don't want to show this to users for this type of Web Part)

Note that one could argue that the stuff specific to the Content Editor Web Part
doesn't really belong in this class, but nevertheless that's where it ended up
after refactoring the code a few years ago on the Agilent Technologies project.
[Actually, I believe it originally ended up in an even more generic
`SharePointHelper` class, but I've since refactored it even further into the
`SharePointWebPartHelper` class.]

Anyway, here's the code for the `SharePointWebPartHelper` class:

```
using System;
using System.Diagnostics;
using System.Globalization;
using System.IO;
using System.Security.Permissions;
using System.Web.UI.WebControls.WebParts;
using System.Xml;

using Microsoft.SharePoint;
using Microsoft.SharePoint.Security;
using SPWebPartPages = Microsoft.SharePoint.WebPartPages;

using Fabrikam.Demo.CoreServices.Logging;

namespace Fabrikam.Demo.CoreServices.SharePoint
{
    /// <summary>
    /// Exposes static methods for commonly used helper functions for
    /// SharePoint Web Parts. This class cannot be inherited.
    /// </summary>
    /// <remarks>
    /// All methods of the <b>SharePointWebPartHelper</b> class are static and
    /// can therefore be called without creating an instance of the class.
    /// </remarks>
    [CLSCompliant(false)]
    [SharePointPermission(SecurityAction.LinkDemand, ObjectModel = true)]
    public sealed class SharePointWebPartHelper
    {
        private SharePointWebPartHelper() { } // all members are static

        private static WebPart CreateWebPart(
            SPWebPartPages.SPLimitedWebPartManager wpm,
            string webPartId,
            string webPartFilename,
            string zoneId,
            ref int zoneIndex)
        {
            Debug.Assert(wpm != null);
            Debug.Assert(string.IsNullOrEmpty(webPartId) == false);
            Debug.Assert(string.IsNullOrEmpty(webPartFilename) == false);
            Debug.Assert(string.IsNullOrEmpty(zoneId) == false);

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Creating Web Part ({0}) on page ({1})...",
                webPartId,
                wpm.ServerRelativeUrl);

            WebPart webPart = null;

            SPFile wpFile = GetWebPartFileFromGallery(
                wpm.Web.Site,
                webPartFilename);

            Debug.Assert(wpFile != null);

            string errorMessage = string.Empty;

            using (Stream stream = wpFile.OpenBinaryStream())
            {
                XmlReader reader = XmlReader.Create(stream);
                webPart = wpm.ImportWebPart(reader, out errorMessage);
            }

            if (string.IsNullOrEmpty(errorMessage) == false)
            {
                string message =
                    "Error importing Web Part ("
                        + webPartFilename + "): "
                        + errorMessage;

                throw new SPWebPartPages.WebPartPageUserException(message);
            }

            webPart.ID = webPartId;
            wpm.AddWebPart(webPart, zoneId, zoneIndex);

            // HACK: When adding the first, second, third, and fourth Web Parts
            // to a zone:
            //
            //    SPLimitedWebPartManager.AddWebPart(webPart, zoneId, zoneIndex)
            //
            // behaves as expected, meaning that
            // (webPart.ZoneIndex == zoneIndex).
            //
            // However, when adding the fifth and sixth Web Parts to a zone,
            // then (webPart.ZoneIndex == zoneIndex + 1).
            if (webPart.ZoneIndex > zoneIndex)
            {
                Debug.Assert(zoneIndex >= 4);

                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "HACK: webPart.ZoneIndex ({0})"
                        + " is greater than zoneIndex ({1})."
                        + " Setting zoneIndex to {0}...",
                    webPart.ZoneIndex,
                    zoneIndex);

                zoneIndex = webPart.ZoneIndex;
            }

            Debug.Assert(webPart.ZoneIndex == zoneIndex);
            zoneIndex++;

            Logger.LogInfo(
                CultureInfo.InvariantCulture,
                "Successfully created Web Part ({0}) on page ({1}).",
                webPartId,
                wpm.ServerRelativeUrl);

            return webPart;
        }

        /// <summary>
        /// Ensures that a specific Web Part exists on a page.
        /// </summary>
        /// <param name="wpm">The Web Part manager for the page.</param>
        /// <param name="webPartId">The unique identifier for the Web Part.</param>
        /// <param name="webPartFileName">The .dwp or .webpart file in the
        /// Web Part gallery (used to import the Web Part if it does not exist).
        /// </param>
        /// <param name="zoneId">The zone the Web Part should reside in.</param>
        /// <param name="zoneIndex">The index of the Web Part relative to other
        /// Web Parts in the zone. If the Web Part is created, the index is
        /// incremented.</param>
        /// <returns>The new or existing Web Part on the page.</returns>
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Design",
            "CA1045:DoNotPassTypesByReference",
            Justification = "The zoneIndex is incremented when the Web Part is"
                + " added.")]
        public static WebPart EnsureWebPart(
            SPWebPartPages.SPLimitedWebPartManager wpm,
            string webPartId,
            string webPartFileName,
            string zoneId,
            ref int zoneIndex)
        {
            if (wpm == null)
            {
                throw new ArgumentNullException("wpm");
            }

            if (string.IsNullOrEmpty(webPartId) == true)
            {
                throw new ArgumentException(
                    "The Web Part Id must be specified.",
                    "webPartId");
            }

            if (string.IsNullOrEmpty(webPartFileName) == true)
            {
                throw new ArgumentException(
                    "The Web Part file name must be specified.",
                    "webPartFileName");
            }

            if (string.IsNullOrEmpty(zoneId) == true)
            {
                throw new ArgumentException(
                    "The Web Part zone must be specified.",
                    "zoneId");
            }

            WebPart webPart = wpm.WebParts[webPartId];

            if (webPart == null)
            {
                webPart = CreateWebPart(wpm, webPartId, webPartFileName,
                    zoneId, ref zoneIndex);
            }
            else
            {
                // HACK: webPart.Zone appears to always be null (SharePoint bug?)
                if (webPart.Zone == null)
                {
                    Logger.LogDebug(
                        CultureInfo.InvariantCulture,
                        "Unable to check zone for Web Part ({0})"
                            + " because webPart.Zone is null.",
                        webPartId);
                }
                else if (webPart.Zone.ID != zoneId)
                {
                    Logger.LogDebug(
                        CultureInfo.InvariantCulture,
                        "Moving Web Part ({0}) to zone ({1})...",
                        webPartId,
                        zoneId);

                    wpm.MoveWebPart(webPart, zoneId, zoneIndex);
                    wpm.SaveChanges(webPart);

                    Logger.LogInfo(
                        CultureInfo.InvariantCulture,
                        "Successfully moved Web Part ({0}) to zone ({1}).",
                        webPartId,
                        zoneId);
                }
            }

            return webPart;
        }

        /// <summary>
        /// Finds the first Web Part with the specified title.
        /// </summary>
        /// <remarks>
        /// This is useful in cases where the Web Part ID is not specified
        /// when it was added to the collection of Web Parts (thus generating
        /// a random identifier and making SPLimitedWebPartCollection[id]
        /// effectively useless).
        /// </remarks>
        /// <param name="wpm">The Web Part manager for the page.</param>
        /// <param name="webPartTitle">The title of the Web Part to find.</param>
        /// <returns>A Web Part object (if one is found), otherwise null.</returns>
        public static WebPart FindWebPartByTitle(
            SPWebPartPages.SPLimitedWebPartManager wpm,
            string webPartTitle)
        {
            if (wpm == null)
            {
                throw new ArgumentNullException("wpm");
            }

            if (string.IsNullOrEmpty(webPartTitle) == true)
            {
                throw new ArgumentException(
                    "The Web Part title must be specified.",
                    "webPartTitle");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Attemping to find Web Part ({0}) on page ({1})...",
                webPartTitle,
                wpm.ServerRelativeUrl);

            for (int i = 0; i < wpm.WebParts.Count; i++)
            {
                WebPart webPart = wpm.WebParts[i];

                if (string.Compare(
                        webPart.Title,
                        webPartTitle,
                        StringComparison.OrdinalIgnoreCase) == 0)
                {
                    Logger.LogDebug(
                        CultureInfo.InvariantCulture,
                        "Found Web Part ({0}) on page ({1}).",
                        webPartTitle,
                        wpm.ServerRelativeUrl);

                    return webPart;
                }
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Unable to find Web Part ({0}) on page ({1}).",
                webPartTitle,
                wpm.ServerRelativeUrl);

            return null;
        }

        private static SPFile GetWebPartFileFromGallery(
            SPSite site,
            string webPartFilename)
        {
            if (site == null)
            {
                throw new ArgumentNullException("site");
            }

            if (webPartFilename == null)
            {
                throw new ArgumentNullException("webPartFilename");
            }

            webPartFilename = webPartFilename.Trim();
            if (string.IsNullOrEmpty(webPartFilename))
            {
                throw new ArgumentException(
                    "The Web Part filename must be specified.",
                    "webPartFilename");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Getting Web Part file ({0}) from site ({1}) gallery...",
                webPartFilename,
                site.Url);

            SPList wpGallery = site.GetCatalog(SPListTemplateType.WebPartCatalog);

            string camlQuery = "<Where><Eq><FieldRef Name=\"FileLeafRef\" />"
                + "<Value Type=\"File\">" + webPartFilename + "</Value></Eq></Where>";

            SPQuery query = new SPQuery();
            query.Query = camlQuery;

            SPListItemCollection webParts = wpGallery.GetItems(query);
            if (webParts.Count == 0)
            {
                string message = string.Format(
                    CultureInfo.InvariantCulture,
                    "Web Part ({0}) was not found in the Web Part Gallery.",
                    webPartFilename);

                throw new ArgumentException(
                    message,
                    "webPartFilename");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Found Web Part file ({0}) in site ({1}) gallery.",
                webPartFilename,
                site.Url);

            Debug.Assert(webParts.Count == 1);
            return webParts[0].File;
        }

        /// <summary>
        /// Ensures that a Content Editor Web Part exists on a page
        /// and is configured with the specified settings.
        /// </summary>
        /// <param name="wpm">The Web Part manager for the page.</param>
        /// <param name="webPartId">The unique identifier for the Web Part.</param>
        /// <param name="zoneId">The zone the Web Part should reside in.</param>
        /// <param name="zoneIndex">The index of the Web Part relative to other
        /// Web Parts in the zone. If the Web Part is created, the index is
        /// incremented.</param>
        /// <param name="htmlContent">The content to display within the Web
        /// Part.</param>
        /// <param name="title">The title of the Web Part.</param>
        /// <returns>The new or existing Web Part on the page.</returns>
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Design",
            "CA1045:DoNotPassTypesByReference",
            Justification="The zoneIndex is incremented when the Web Part is"
                + " added.")]
        public static SPWebPartPages.ContentEditorWebPart
            EnsureContentEditorWebPart(
                SPWebPartPages.SPLimitedWebPartManager wpm,
                string webPartId,
                string zoneId,
                ref int zoneIndex,
                string htmlContent,
                string title)
        {
            if (wpm == null)
            {
                throw new ArgumentNullException("wpm");
            }

            if (webPartId == null)
            {
                throw new ArgumentNullException("webPartId");
            }

            webPartId = webPartId.Trim();
            if (string.IsNullOrEmpty(webPartId))
            {
                throw new ArgumentException(
                    "The Web Part ID must be specified.",
                    "webPartId");
            }

            if (zoneId == null)
            {
                throw new ArgumentNullException("zoneId");
            }

            zoneId = zoneId.Trim();
            if (string.IsNullOrEmpty(zoneId))
            {
                throw new ArgumentException(
                    "The Web Part zone must be specified.",
                    "zoneId");
            }

            if (htmlContent == null)
            {
                throw new ArgumentNullException("htmlContent");
            }

            htmlContent = htmlContent.Trim();
            if (string.IsNullOrEmpty(htmlContent))
            {
                throw new ArgumentException(
                    "The HTML content must be specified.",
                    "htmlContent");
            }

            // Note: title is optional

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Ensuring Content Editor Web Part ({0}) on page ({1})...",
                webPartId,
                wpm.ServerRelativeUrl);

            SPWebPartPages.ContentEditorWebPart webPart =
                (SPWebPartPages.ContentEditorWebPart) EnsureWebPart(
                    wpm,
                    webPartId,
                    "MSContentEditor.dwp",
                    zoneId,
                    ref zoneIndex);

            if (webPart.Content == null
                || webPart.Content.InnerText != htmlContent
                || webPart.Title != title
                || webPart.ChromeType != PartChromeType.None)
            {
                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "Updating properties of Content Editor Web Part ({0}) on"
                        + " page ({1})...",
                    webPartId,
                    wpm.ServerRelativeUrl);

                // To bring content into the Content Editor Web Part,
                // we need to use an XML Document
                XmlDocument xmlDoc = new XmlDocument();
                XmlElement xmlElement = xmlDoc.CreateElement("HtmlContent");
                xmlElement.InnerText = htmlContent;
                webPart.Content = xmlElement;

                webPart.Title = title;
                webPart.ChromeType = PartChromeType.None;

                wpm.SaveChanges(webPart);

                Logger.LogInfo(
                    CultureInfo.InvariantCulture,
                    "Successfully updated properties of Content Editor Web Part"
                        + " ({0}) on page ({1}).",
                    webPartId,
                    wpm.ServerRelativeUrl);
            }
            else
            {
                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "The Content Editor Web Part ({0}) on page ({1}) is already"
                        + " configured with the expected settings.",
                    webPartId,
                    wpm.ServerRelativeUrl);
            }

            return webPart;
        }
    }
}
```

Most of the code in the `SharePointWebPartHelper` class is very straightforward.
The most interesting hack -- er, I mean *workaround* -- in the code is to handle
the scenario where the `zoneIndex` is sometimes mysteriously incremented by more
than one, but you can read more about this in a
[previous post](/blog/jjameson/2009/06/05/splimitedwebpartmanager-addwebpart-mysteriously-increments-zoneindex)
-- if you are really interested.

Using the `SharePointWebPartHelper` class couldn't be easier. For a simple
example, consider the scenario that I mentioned earlier about adding a Login
Form Web Part to a page:

```
            // Configure Web Parts
            SPWebPartPages.SPLimitedWebPartManager wpm =
                publicWeb.GetLimitedWebPartManager(
                    page.Url,
                    PersonalizationScope.Shared);

            using (wpm)
            {
                string zoneId = "LeftColumnZone";
                int zoneIndex = 0;

                LoginFormWebPart loginForm =
                    (LoginFormWebPart) SharePointWebPartHelper.EnsureWebPart(
                        wpm,
                        "LoginForm",
                        "Fabrikam_LoginForm.webpart",
                        zoneId,
                        ref zoneIndex);

                loginForm.DisplayRememberMe = false;
                wpm.SaveChanges(loginForm);

                // HACK: Avoid memory leak in SPLimitedWebPartManager
                if (wpm.Web != null)
                {
                    wpm.Web.Dispose();
                }
            }
```

Just be sure to avoid
[the memory leak in SPLimitedWebPartManager](/blog/jjameson/2008/04/09/memory-leak-in-splimitedwebpartmanager-a-k-a-idisposables-containing-idisposables),
like I show in the code example above.

Here's another code example where I configured a bunch of Web Parts on a search
results page:

```
namespace Fabrikam.Web.Search.Configuration
{
    /// <summary>
    /// Exposes static methods for configuring the <b>Fabrikam.Web.Search</b>
    /// feature. This class cannot be inherited.
    /// </summary>
    /// <remarks>
    /// All methods of the <b>FeatureConfigurator</b> class are static and can
    /// therefore be called without creating an instance of the class.
    /// </remarks>
    [CLSCompliant(false)]
    [SharePointPermission(SecurityAction.LinkDemand, ObjectModel = true)]
    public sealed class FeatureConfigurator
    {
        ...

        private static void ConfigureSearchResultsPageWebParts(
            SPWeb searchWeb,
            PublishingPage searchResultsPage,
            string defaultSearchResultsPageUrl,
            string scopeName,
            string additionalQueryTerms,
            string contextualScopeUrl)
        {
            SPWebPartPages.SPLimitedWebPartManager wpm =
                searchWeb.GetLimitedWebPartManager(
                    searchResultsPage.Url, PersonalizationScope.Shared);

            using (wpm)
            {
                #region Zone1

                string zoneId = "Zone1";
                int zoneIndex = 0;

                SharePointWebPartHelper.EnsureContentEditorWebPart(
                    wpm,
                    "startNewSearchHeading",
                    zoneId,
                    ref zoneIndex,
                    "<h2 class='accent3'>Start a new search</h2>",
                    "Start New Search Heading");

                SearchBoxEx searchBox =
                    (SearchBoxEx)SharePointWebPartHelper.EnsureWebPart(
                        wpm,
                        "searchBox",
                        "SearchBox.dwp",
                        zoneId,
                        ref zoneIndex);

                searchBox.DropDownModeEx = DropDownModesEx.HideScopeDD;
                searchBox.SearchResultPageURL = defaultSearchResultsPageUrl;
                searchBox.Width = string.Empty;

                wpm.SaveChanges(searchBox);

                SharePointWebPartHelper.EnsureContentEditorWebPart(
                    wpm,
                    "refineSearchHeading",
                    zoneId,
                    ref zoneIndex,
                    "<h2 class='accent3'>Refine your search</h2>",
                    "Refine Search Heading");

                FabrikamSearchBoxWebPart searchScopeOptions =
                    (FabrikamSearchBoxWebPart)SharePointWebPartHelper.EnsureWebPart(
                        wpm,
                        "searchScopeOptions",
                        "FabrikamSearchBox.webpart",
                        zoneId,
                        ref zoneIndex);

                searchScopeOptions.ShowSearchBox = false;
                wpm.SaveChanges(searchScopeOptions);

                ConfigureFacetedSearchWebParts(
                    wpm,
                    zoneId,
                    ref zoneIndex,
                    scopeName);

                #endregion

                #region Zone2

                zoneId = "Zone2";
                zoneIndex = 0;

                SharePointWebPartHelper.EnsureWebPart(
                    wpm,
                    "searchTabs",
                    "FabrikamListBoundTabStrip.webpart",
                    zoneId,
                    ref zoneIndex);

                SharePointWebPartHelper.EnsureWebPart(
                    wpm,
                    "searchSummary",
                    "SearchSummary.dwp",
                    zoneId,
                    ref zoneIndex);

                bool showBestBetsWebPart = false;

                if (string.Compare(
                    searchResultsPage.Name,
                    "Results.aspx",
                    StringComparison.OrdinalIgnoreCase) == 0)
                {
                    showBestBetsWebPart = true;
                }
                else if (string.Compare(
                    searchResultsPage.Name,
                    "AllSitesResults.aspx",
                    StringComparison.OrdinalIgnoreCase) == 0)
                {
                    showBestBetsWebPart = true;
                }

                if (showBestBetsWebPart == true)
                {
                    ConfigureBestBetsWebPartForSiteSearch(
                        wpm,
                        zoneId,
                        ref zoneIndex);
                }

                SearchStatsWebPart searchStats =
                    (SearchStatsWebPart) SharePointWebPartHelper.EnsureWebPart(
                        wpm,
                        "searchStats",
                        "searchStats.dwp",
                        zoneId,
                        ref zoneIndex);

                searchStats.DisplayResponseTime = false;
                wpm.SaveChanges(searchStats);

                SharePointWebPartHelper.EnsureWebPart(
                    wpm,
                    "searchPaging1",
                    "SearchPaging.dwp",
                    zoneId,
                    ref zoneIndex);

                ConfigureCoreResultsWebPartsForSiteSearch(
                    wpm,
                    zoneId,
                    ref zoneIndex,
                    scopeName);

                SharePointWebPartHelper.EnsureWebPart(
                    wpm,
                    "searchPaging2",
                    "SearchPaging.dwp",
                    zoneId,
                    ref zoneIndex);

                if (additionalQueryTerms != null
                    || contextualScopeUrl != null)
                {
                    ConfigureCoreResultsExtensionWebPart(
                        wpm,
                        additionalQueryTerms,
                        contextualScopeUrl,
                        zoneId,
                        ref zoneIndex);
                }

                #endregion

                // HACK: Avoid memory leak in SPLimitedWebPartManager
                if (wpm.Web != null)
                {
                    wpm.Web.Dispose();
                }
            }
        }
    }
}
```

Even though I didn't add any comments to this (rather lengthy) method, I think
you'll find it is actually very easy to understand simply by reading the method
names and parameters -- or at least that's what I hope for the people that are
actually maintaining this code from my previous project ;-)

I hope you find the `SharePointPublishingHelper` and `SharePointWebPartHelper`
classes as useful as I have over the past several years.

Be sure to stay tuned, because there are more SharePoint helper classes to come,
but my daughter just woke up and now I need to go make some pancakes!
