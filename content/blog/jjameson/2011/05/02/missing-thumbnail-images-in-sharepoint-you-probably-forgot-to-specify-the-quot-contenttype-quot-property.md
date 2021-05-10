---
title: "Missing thumbnail images in SharePoint?...You probably forgot to specify the \"ContentType\" property"
date: 2011-05-02T05:33:00-06:00
description:
  During the process of creating my previous post , I discovered the thumbnail
  images were not rendering as expected for the custom images that I added to
  the out-of-the-box SharePoint /PublishingImages picture library (via a
  feature). Here is the content...
aliases:
  [
    "/blog/jjameson/archive/2011/05/01/missing-thumbnail-images-in-sharepoint-you-probably-forgot-to-specify-the-quot-contenttype-quot-property.aspx",
    "/blog/jjameson/archive/2011/05/02/missing-thumbnail-images-in-sharepoint-you-probably-forgot-to-specify-the-quot-contenttype-quot-property.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007", "SharePoint 2010"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2011/05/02/missing-thumbnail-images-in-sharepoint-you-probably-forgot-to-specify-the-quot-contenttype-quot-property.aspx"
---

During the process of creating
[my previous post](/blog/jjameson/2011/05/02/web-standards-design-with-sharepoint-part-6),
I discovered the thumbnail images were not rendering as expected for the custom
images that I added to the out-of-the-box SharePoint /PublishingImages picture
library (via a feature).

Here is the content of my original Elements.xml file (used to add the images to
the library):

```XML
<?xml version="1.0" encoding="utf-8" ?>
<Elements xmlns="http://schemas.microsoft.com/sharepoint/">
  <Module
    Name="HomeSiteImages"
    Url="PublishingImages"
    Path="HomeSiteConfiguration\PublishingImages">
    <File Url="boat.jpg" Type="GhostableInLibrary"/>
    <File Url="fame.jpg" Type="GhostableInLibrary"/>
    <File Url="ropes.jpg" Type="GhostableInLibrary"/>
  </Module>
</Elements>
```

To get the thumbnails to render as expected, I simply modified the Elements.xml
file as follows:

```XML
<?xml version="1.0" encoding="utf-8" ?>
<Elements xmlns="http://schemas.microsoft.com/sharepoint/">
  <Module
    Name="HomeSiteImages"
    Url="PublishingImages"
    Path="HomeSiteConfiguration\PublishingImages">
    <File Url="boat.jpg" Type="GhostableInLibrary">
      <Property Name="ContentType"
        Value="$Resources:cmscore,contenttype_image_name;" />
    </File>
    <File Url="fame.jpg" Type="GhostableInLibrary">
      <Property Name="ContentType"
        Value="$Resources:cmscore,contenttype_image_name;" />
    </File>
    <File Url="ropes.jpg" Type="GhostableInLibrary">
      <Property Name="ContentType"
        Value="$Resources:cmscore,contenttype_image_name;" />
    </File>
  </Module>
</Elements>
```

By simply adding the **ContentType** property (and setting it to "Image"),
SharePoint automatically creates the corresponding thumbnail for each image when
adding the item to the picture library.
