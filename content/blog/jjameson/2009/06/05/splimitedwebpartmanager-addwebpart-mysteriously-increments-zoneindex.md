---
title: SPLimitedWebPartManager.AddWebPart Mysteriously Increments zoneIndex
date: 2009-06-05T08:08:00-06:00
excerpt:
  "One of the common tasks when using the \"DR.DADA\" approach to SharePoint
  development is programmatically creating and configuring pages on a site. This
  often requires adding numerous Web Parts to various zones on a page -- for
  example, to configure search..."
aliases:
  [
    "/blog/jjameson/archive/2009/06/04/splimitedwebpartmanager-addwebpart-mysteriously-increments-zoneindex.aspx",
    "/blog/jjameson/archive/2009/06/05/splimitedwebpartmanager-addwebpart-mysteriously-increments-zoneindex.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/06/05/splimitedwebpartmanager-addwebpart-mysteriously-increments-zoneindex.aspx"
---

One of the common tasks when using the
["DR.DADA" approach to SharePoint development](/blog/jjameson/2009/03/31/introducing-the-dr-dada-approach-to-sharepoint-development)
is programmatically creating and configuring pages on a site. This often
requires adding numerous Web Parts to various zones on a page -- for example, to
configure search results pages.

Over the past couple of years, my teammates and I have created numerous "helper"
classes that simplify this task in Windows SharePoint Services (WSS) 3.0 and
Microsoft Office SharePoint Server (MOSS) 2007 -- including
`SharePointWebPartHelper`.

```C#
/// <summary>
/// Exposes static methods for commonly used helper functions for
/// SharePoint Web Parts. This class cannot be inherited.
/// </summary>
/// <remarks>
/// All methods of the <b>SharePointWebPartHelper</b> class are static and
/// can therefore be called without creating an instance of the class.
/// </remarks>    
public sealed class SharePointWebPartHelper
```

`SharePointWebPartHelper` contains public methods like `EnsureWebPart` and
`FindWebPartByTitle`. As you can probably tell from the names of these methods,
this allows you to locate an existing Web Part on a page or create one if the
specified Web Part doesn't currently exist on the page.

`EnsureWebPart` attempts to find a Web Part based on the specified webPartId and
if the Web Part is not found, then it uses the `CreateWebPart` method to create
a new Web Part. Here is the original implementation of the `CreateWebPart`
method:

```C#
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

            throw new ApplicationException(message);
        }

        webPart.ID = webPartId;
        wpm.AddWebPart(webPart, zoneId, zoneIndex);        
        zoneIndex++;
        
        Logger.LogDebug(
            CultureInfo.InvariantCulture,
            "Successfully created Web Part ({0}) on page ({1}).",
            webPartId,
            wpm.ServerRelativeUrl);

        return webPart;
    }
```

Note that `SPWebPartPages` is simply an alias for the
`Microsoft.SharePoint.WebPartPages` namespace (in order to avoid "collisions"
with the `System.Web.UI.WebControls.WebParts` namespace). Also, please ignore
the use of `ApplicationException` -- apparently there's still a little "code
cleanup" to be done here.

As I called out above, this code shows the _original_ implementation of the
method. It turns out there's a very obscure bug in it, that I only discovered
within the last couple of months.

As you can probably guess from the title of this post, the problem is that
`zoneIndex` (i.e. the position of the Web Part relative to other Web Parts
within the same Web Part zone) isn't always incremented as you would expect.

When adding the first, second, third, and fourth Web Parts to a zone:

```C#
    SPLimitedWebPartManager.AddWebPart(webPart, zoneId, zoneIndex)
```

behaves as expected, meaning that (`webPart.ZoneIndex == zoneIndex`).

However, when adding the fifth and sixth Web Parts to a zone, then
(`webPart.ZoneIndex == zoneIndex + 1`).

I suspect it is due to the `SPWebPartManager.MakeSpaceForWebpart` method (which
is called from `SPLimitedWebPartManager.AddWebPart`). However, the
`SPWebPartManager.MakeSpaceForWebpart` method is obfuscated and therefore I
can't see what it is doing using Reflector.

This behavior caused some Web Parts to appear in the wrong location, and
therefore I had to come up with a hack for it. Here is my updated version of the
`CreateWebPart` method:

```C#
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

            throw new ApplicationException(message);
        }

        webPart.ID = webPartId;
        wpm.AddWebPart(webPart, zoneId, zoneIndex);

        // Bug 68288:
        //
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
        
        Logger.LogDebug(
            CultureInfo.InvariantCulture,
            "Successfully created Web Part ({0}) on page ({1}).",
            webPartId,
            wpm.ServerRelativeUrl);

        return webPart;
    }
```

I was curious to know why SharePoint behaves this way and to see if this
behavior is documented somewhere and I just missed it. Unfortunately, the e-mail
I sent back in March to an internal discussion list received no responses. So I
guess this will just have to remain a mystery.

