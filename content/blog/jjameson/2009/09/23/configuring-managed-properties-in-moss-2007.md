---
title: Configuring Managed Properties in MOSS 2007
date: 2009-09-23T07:09:00-06:00
description:
  "As I've noted in a previous post , I typically use feature receivers in
  Microsoft Office SharePoint Server (MOSS) 2007 to automatically configure a
  \"bunch of stuff\" that would otherwise be very tedious to perform repeatedly
  for different environments..."
aliases:
  [
    "/blog/jjameson/archive/2009/09/22/configuring-managed-properties-in-moss-2007.aspx",
    "/blog/jjameson/archive/2009/09/23/configuring-managed-properties-in-moss-2007.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/09/23/configuring-managed-properties-in-moss-2007.aspx"
---

As I've noted in a
[previous post](/blog/jjameson/2009/03/31/introducing-the-dr-dada-approach-to-sharepoint-development),
I typically use feature receivers in Microsoft Office SharePoint Server (MOSS)
2007 to automatically configure a "bunch of stuff" that would otherwise be very
tedious to perform repeatedly for different environments (e.g. DEV, TEST, and
PROD) and whenever I rebuild my local development VM. For example, when you
activate one of my "Search" features, I typically create the Search Center,
create/configure the various search results pages with a number of different Web
Parts, and also programmatically configure managed properties.

For this post, I want to focus on the managed properties. In case you are not
familiar with crawled properties and managed properties in MOSS 2007, let me
start by providing a little background.

When SharePoint indexes content:

1. It first starts up a _protocol handler_ for the content source (e.g.
   SharePoint site, file system, Lotus Notes DB, etc.). The protocol handler is
   responsible for enumerating content items in the content source.
1. For each content item, it then loads an _IFilter_ (e.g. HTML, Office doc,
   PDF, etc). The IFilter is responsible for emitting text and properties from
   the underlying content item.
1. These properties (e.g. Author) are then picked up as _crawled properties_.
1. For custom columns in SharePoint list items and documents (e.g. **Product**),
   the crawled properties are discovered and placed in the Office category (e.g.
   **ows_Product**). [If memory serves, "ows" refers to "Office Web Server" (the
   original moniker for what ultimately became "SharePoint Products and
   Technologies") -- if that helps you remember this any easier.]
1. If any _managed properties_ are mapped to the crawled properties, then the
   property values are stuffed into the SSP Search database (i.e. what used be
   called the "property store" in SharePoint Portal Server 2003) for each piece
   of content.

While SharePoint comes with about 110+ managed properties OOTB, customers
typically add new ones -- especially when providing any sort of
[faceted search](/blog/jjameson/2009/09/18/faceted-search-in-moss-2007-and-the-mssdocprops-issue)
feature.

So, for example, suppose we want to filter search results on **Product**. First
we must define a managed property.

Once you define a managed property, you must map it to a crawled property
(otherwise the managed property values would always be empty). After configuring
managed properties, you must perform a full crawl in order to populate the
properties.

The problem is that you cannot map a managed property to a crawled property
until SharePoint actually knows about the crawled property.

In other words, if you start with a clean MOSS environment and then add a
**Product** managed property, you will find that you cannot map it to
**ows_Product** because SharePoint doesn't yet know about that crawled property.

In order for it to recognize the crawled property, it must first have indexed a
piece of content with a value specified in the **Product** field (column). Then
SharePoint will create a crawled property (as mentioned earlier).

I also mentioned earlier that upon activation of my Search features, I
automatically configure managed properties (and map them to the corresponding
crawled properties). Assuming the crawled properties are defined (i.e. content
has previously been crawled with those properties), then all is well. However,
if the specified crawled property does not exist, then SharePoint silently
ignores the attempt to map the managed property to the (non-existent) crawled
property.

Hence, you need to ensure that a piece of content has all custom properties
specified, then perform a full crawl (to recognize all of the crawled
properties), and then finally deactivate and reactivate my custom Search feature
to successfully map the managed properties to the crawled properties. Then you
must run another full crawl (in order to actually populate the managed
properties).

I often refer to this as "the chicken and the egg" problem with SharePoint
managed properties. That's certainly not the official name for this issue, but
it seems to help people understand why the managed properties are not mapped
after activating the custom Search feature in a freshly rebuilt SharePoint
environment.

The "magic" behind automatically configuring managed properties upon feature
activation is really not magic at all. It simply uses my
"[FeatureConfigurator](/blog/jjameson/2007/03/22/what-s-in-a-name-defaultfeaturereceiver-vs-featureconfigurator)
framework" (and even calling this a framework is definitely a stretch) with a
little bit of help from my SharePointSearchHelper class:

```C#
namespace Fabrikam.Project1.Search.Configuration
{
    /// <summary>
    /// Exposes static methods for configuring the <b>Fabrikam.Project1.Search</b>
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

        /// <summary>
        /// Configures the various objects used for the Search features,
        /// including custom search scopes and managed properties, as well as a
        /// Search Center (and all of the corresponding pages) on the
        /// specified site.
        /// </summary>
        /// <param name="web">An
        /// <see cref="Microsoft.SharePoint.SPSite"/> object representing the
        /// site collection to configure.</param>
        public static void Configure(
            SPSite site)
        {
            if (site == null)
            {
                throw new ArgumentNullException("site");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Configuring Fabrikam.Project1.Search on site ({0})...",
                site.Url);

            ConfigureSspSettings(site);

            ConfigureSearchCenter(site);

            Logger.LogInfo(
                CultureInfo.InvariantCulture,
                "Successfully configured Fabrikam.Project1.Search on site ({0}).",
                site.Url);
        }

        ...

        private static void ConfigureManagedProperties(
            Schema sspSchema)
        {
            Debug.Assert(sspSchema != null);

            ManagedPropertyCollection properties = sspSchema.AllManagedProperties;

            // Configure "ListItemIntId" managed property
            ManagedProperty property =
                SharePointSearchHelper.EnsureManagedProperty(
                    properties,
                    "ListItemIntId",
                    ManagedDataType.Integer);

            SharePointSearchHelper.EnsureManagedPropertyMapping(
                property,
                "ows_ID",
                SharePointSearchHelper.CrawledPropertyCategory.SharePoint,
                SharePointSearchHelper.CrawledPropertyVariantType.Integer);

            // Configure "Product" managed property
            property = SharePointSearchHelper.EnsureManagedProperty(
                properties,
                "Product",
                ManagedDataType.Text);

            SharePointSearchHelper.EnsureManagedPropertyMapping(
                property,
                "ows_Product",
                SharePointSearchHelper.CrawledPropertyCategory.SharePoint,
                SharePointSearchHelper.CrawledPropertyVariantType.MultiValuedText);

            // Configure "ContentTypeName" managed property
            ...
        }

        ...

        private static void ConfigureSspSettings(
            SPSite site)
        {
            Debug.Assert(site != null);

            SearchContext context = SearchContext.GetContext(site);
            if (context == null)
            {
                string message = string.Format(
                    CultureInfo.InvariantCulture,
                    "Unable to get SearchContext for site ({0}).",
                    site.Url);

                throw new ArgumentException(message);
            }

            Schema sspSchema = new Schema(context);
            if (sspSchema == null)
            {
                string message = string.Format(
                    CultureInfo.InvariantCulture,
                    "Unable to get SSP schema for search context ({0}).",
                    context.Name);

                throw new ArgumentException(message);
            }

            ConfigureFileTypes(context);

            ConfigureManagedProperties(sspSchema);

            ConfigureSearchScopes(context, site.Url);
        }

        ...
    }
}
```
