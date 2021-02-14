---
title: "No HTML Markup in MOSS 2007 Managed Properties"
date: 2009-09-23T02:35:00+08:00
excerpt: "In my previous post , I showed how to automatically configure managed properties in Microsoft Office SharePoint Server (MOSS) 2007 when activating a custom \"Search\" feature. In this post, I want to cover a subtle, yet very important, limitation in managed..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:  
>   
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/09/23/no-html-markup-in-moss-2007-managed-properties.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/09/23/no-html-markup-in-moss-2007-managed-properties.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/archive/2011/09/02/last-day-with-microsoft.aspx), I have copied it here in case that blog ever goes away.


In my [previous post](/blog/jjameson/archive/2009/09/23/configuring-managed-properties-in-moss-2007.aspx), I showed how to automatically configure managed properties in Microsoft Office SharePoint Server (MOSS) 2007 when activating a custom "Search" feature. In this post, I want to cover a subtle, yet very important, limitation in managed properties.

Suppose that you have created a custom field (column) called **Abstract** that allows users to specify formatting (i.e. HTML markup). Furthermore, suppose that you want to display **Abstract **for each search result, instead of, say, the HitHighlightedSummary. Therefore, you create a new **Abstract **managed property, map it to **ows\_Abstract**, and subsequently perform a full crawl to populate the new managed property. You then modify the properties of the Search Core Results Web Part to include **Abstract **in the list of columns returned in search results and also modify the XSLT to display the value of **Abstract** instead of HitHighlightedSummary.

The problem is that any formatting (i.e. HTML markup) specified by users in **Abstract **is not displayed in search results. When I contacted the SharePoint team about this a few years ago (while working on the Agilent Technologies project), I was told that this was "by design" (i.e., there are no plans to change this behavior). In other words, when indexing content, SharePoint intentionally strips any HTML markup from the property values when populating the managed properties. [Whether this happens in the IFilter when emitting properties or actually within the SharePoint crawler really doesn't matter; the fact is it just does it.]

Note that this is important not only for fields that allow HTML, but also for fields such as PublishingRollupImage (i.e. [Microsoft.SharePoint.Publishing.Fields.ImageFieldValue](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.publishing.fields.imagefieldvalue.aspx)). For example, suppose you wanted to show the "rollup image" for each item in search results similar to the Content Query Web Part. You might think that you can simply create a new managed property and map it to the **ows\_PublishingRollupImage **crawled property and then perform similar steps as described above (for **Abstract**) in order to show the images.

Unfortunately, PublishingRollupImage essentially equates to an `<img>` HTML element, which is comprised entirely of markup, and therefore entirely stripped of content when crawled by SharePoint. In other words, the managed property is always null (even though the underlying pages specify a rollup image).

To workaround this issue, I recommend creating a separate field (column) called **PublishingRollupImageUrl** (with a display name of **Rollup Image URL**) and using an [event receiver](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.spitemeventreceiver.aspx) to automatically set this field whenever an item is updated (by extracting the image URL from the PublishingRollupImage field).


    public override void ItemUpdating(
                SPItemEventProperties properties)
            {
                SPListItem updatedPage = properties.ListItem;
    
                // Update PublishingRollupImageUrl (a simple Text field) based on
                // PublishingRollupImage (a Publishing Image field).
                // Note that Publishing Image fields are not compatible with
                // SharePoint Search (i.e. attempting to map a managed property
                // to a Publishing Image field results in a managed property
                // that is always empty).
                ImageFieldValue searchImage =
                    (ImageFieldValue)updatedPage["Rollup Image"];
    
                if (searchImage == null)
                {
                    Logger.LogInfo(
                        CultureInfo.InvariantCulture,
                        "Setting PublishingRollupImageUrl on page ({0}/{1})"
                            + " to null because no rollup image is specified.",
                        updatedPage.Web.Url,
                        updatedPage.Url);
    
                    updatedPage["Rollup Image URL"] = null;
                }
                else
                {
                    Logger.LogInfo(
                        CultureInfo.InvariantCulture,
                        "Setting PublishingRollupImageUrl on page ({0}/{1})"
                            + " to '{2}'.",
                        updatedPage.Web.Url,
                        updatedPage.Url,
                        searchImage.ImageUrl);
    
                    updatedPage["Rollup Image URL"] = searchImage.ImageUrl;
                }
    
                try
                {
                    this.DisableEventFiring();
                    updatedPage.SystemUpdate(false);
                }
                finally
                {
                    this.EnableEventFiring();
                }
            }


You can then create the **PublishingRollupImage** managed property, map it to **ows\_PublishingRollupImageUrl**, and subsequently display the rollup images in search results similar to the Content Query Web Part.

